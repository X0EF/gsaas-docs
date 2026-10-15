---
title: Billing
weight: -15
---

# Billing with Stripe

Code: `billing/`, `adapters/stripe/`, `adapters/postgres/billing.go`

Each organization has its own Stripe customer and subscription. Users start checkout from your app, Stripe sends the result to a webhook, and gsaas keeps the organization's subscription in your database in sync.

## Keys and environment variables

Set these in `.env` (all settings are listed in `.env.example`):

| Variable | What it is |
| --- | --- |
| `STRIPE_SECRET_KEY` | Your Stripe secret key (Dashboard → Developers → API keys). Use a test key until launch. |
| `STRIPE_WEBHOOK_SECRET` | The signing secret of your webhook endpoint (`whsec_…`). Required: unsigned or wrongly signed events are rejected. |
| `STRIPE_PRICE_PRO` | Price ID (`price_…`) of your recurring plan. Checkout plan `pro` uses it. |
| `STRIPE_PRICE_ITEM` | Optional price ID of a one-time purchase. Checkout plan `item` uses it, in payment mode. |
| `FRONTEND_BASE_URL` | Where users land after checkout or the billing portal. Empty uses `APP_BASE_URL`. |
| `CHECKOUT_SUCCESS_PATH`, `CHECKOUT_CANCEL_PATH`, `PORTAL_RETURN_PATH` | Paths on that URL for each outcome. |

Without `STRIPE_SECRET_KEY` the app still boots. Billing routes answer `503` until it is set.

## Plans and prices

1. In Stripe, create a product with a recurring price for your paid plan, and copy its price ID into `STRIPE_PRICE_PRO`.
2. Optionally create a one-time price for `STRIPE_PRICE_ITEM`.
3. Every organization without a subscription is on the free plan.

To add more plans, extend `billing/plan.go` and the plan lookup in `billing/service.go`.

Gate a paid feature with the `RequirePlan` middleware from the billing handler. Organizations without the plan get `402`.

## The webhook

**Events to select** in Stripe (these are the ones gsaas handles; others are ignored):

- `checkout.session.completed`
- `customer.subscription.created`
- `customer.subscription.updated`
- `customer.subscription.deleted`

**Local development:**

```bash
stripe listen --forward-to localhost:5000/webhooks/stripe
```

`stripe listen` prints a `whsec_…` secret. Put it in `STRIPE_WEBHOOK_SECRET`. Use your `PORT` if it isn't 5000.

**Production:** in Dashboard → Developers → Webhooks, add the endpoint `https://<your-api-host>/webhooks/stripe` with the events above, and set its signing secret as `STRIPE_WEBHOOK_SECRET`.

Run the worker (`go run ./cmd/worker`) in every environment. Events are processed as background jobs.

## What happens on each event

```
Stripe ── POST /webhooks/stripe ──> verify signature
                                         │
                                 store event (deduped by Stripe event id)
                                         │
                                 enqueue job ──> worker
                                                   │
                     find org (checkout reference or Stripe customer)
                                                   │
                  update subscription row, or record one-time payment
                                                   │
                           publish subscription.changed
```

- **Signature:** checked with `STRIPE_WEBHOOK_SECRET`. A bad signature gets `401`.
- **Dedupe:** each Stripe event is stored once. A replayed event is a no-op.
- **Retries:** processing runs on the job queue and is retried on failure.
- **What syncs:** the org's subscription status, price, Stripe subscription ID and current period end. One-time purchases are stored as payments.
- **Deleted orgs:** an event for an organization or customer that no longer exists is marked handled and skipped.

## Checkout and portal

Owners and admins of an organization call:

- `POST /organizations/{orgID}/billing/checkout` with `{"plan":"pro"}` (or `"item"`), which returns a Stripe Checkout URL.
- `POST /organizations/{orgID}/billing/portal`, which returns a Stripe customer portal URL. Enable the customer portal in your Stripe settings first.

These routes need a signed-in user. API keys can't use them.

## Reacting to changes

After a subscription change is saved, gsaas publishes the `subscription.changed` event with the org ID and new status. Subscribe to it in `app/subscribe.go` to send emails, unlock features or notify your own systems:

```go
events.Subscribe(bus, billing.EventSubscriptionChanged, func(ctx context.Context, e events.Event[billing.SubscriptionChanged]) error {
	// e.Payload.OrgID, e.Payload.Status
	return nil
})
```
