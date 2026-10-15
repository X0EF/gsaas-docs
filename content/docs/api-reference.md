---
title: API reference
weight: -7
---

# API reference

The running API serves its full reference:

- `GET /docs`: interactive reference (Scalar), e.g. `http://localhost:5000/docs`
- `GET /openapi.yaml`: the OpenAPI 3.1 spec, for clients and code generators

Route groups at a glance:

| Prefix | Auth | Guide |
| --- | --- | --- |
| `/healthz`, `/readyz` | none | [Deployment]({{< relref "deployment" >}}) |
| `/auth` | none, rate limited | [Authentication]({{< relref "authentication" >}}) |
| `/invites/accept` | none, rate limited | [Organizations & multi-tenancy]({{< relref "organizations" >}}) |
| `/me`, `/users` | session | [Authentication]({{< relref "authentication" >}}) |
| `/files` | session | [File uploads]({{< relref "files" >}}) |
| `/organizations` | session or API key | [Organizations & multi-tenancy]({{< relref "organizations" >}}) |
| `/organizations/{orgID}/billing` | session | [Billing]({{< relref "billing" >}}) |
| `/organizations/{orgID}/webhooks` | session or API key | [Webhooks]({{< relref "webhooks" >}}) |
| `/projects` | session or API key, tenant | [Project structure]({{< relref "architecture" >}}) |
| `/jobs` | session or API key, tenant | [Background jobs]({{< relref "jobs" >}}) |
| `/webhooks/stripe` | Stripe signature | [Billing]({{< relref "billing" >}}) |

## Conventions

- Send `Authorization: Bearer <token>`, and `X-Org-ID` on tenant routes without `{orgID}`.
- Responses are `{"object", "data"}`. Lists add `has_more` and `starting_after` for pagination.
- Errors are `{"object": "error", "error": {"type", "message"}}` with a matching HTTP status.
