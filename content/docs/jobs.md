---
title: Background jobs
weight: -12
---

# Background jobs

Slow, retryable or status-visible work runs as a durable job in a Postgres-backed job queue. `platform/queue` defines the interface.

## Running the worker

```bash
go run ./cmd/worker
```

The worker runs alongside the API server. `make migrate` creates the queue tables.

## Behavior

- Jobs enqueued inside a database transaction exist only if it commits.
- A handler that returns an error is retried. Wrap `queue.ErrPermanent` to stop retries.
- Handlers must honor `ctx` and be idempotent.

Built-in jobs include `billing.process_webhook`, `billing.cancel_subscription`, `identity.expire_sessions`, `webhooks.deliver` and the sample `projects.apply`.

## Jobs API

| Route | Permission |
| --- | --- |
| `GET /jobs` | job read |
| `GET /jobs/{id}` | job read |
| `POST /jobs/{id}/cancel` | job write |

The routes are scoped to an organization. System jobs, enqueued without an organization, aren't listed.

## Adding a job

1. In the owning module, export a job name constant and a method `(ctx, payload []byte) error`.
2. Enqueue it through the injected `queue.Queue`.
3. In `RunWorker` in `app/app.go`, map the constant to the method.

In tests, `queue.NewMem` records enqueues.
