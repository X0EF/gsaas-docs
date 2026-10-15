---
title: Project structure
weight: -18
---

# Project structure

Code: `app/`, plus one top-level package per module

```
gsaas/
├── app/            composition root: wiring, routes, event subscriptions, jobs
├── identity/       users, sign-in, sessions, magic links
├── organization/   organizations, members, invites, roles, API keys
├── billing/        plans, checkout, subscriptions
├── projects/       sample module to copy for your own features
├── webhooks/       outbound webhooks
├── files/          file uploads
├── jobs/           job status, list and cancel API
├── platform/       shared helpers: auth, config, email, events, httpx, queue, log
├── adapters/       Postgres, Stripe, Supabase, job queue, S3 storage
├── database/       migrations, SQL queries, generated sqlc code
└── cmd/            server, worker, migrate, seed
```

## Modules

Each module (domain) is a top-level package with the same shape:

| File | Contents |
| --- | --- |
| `service.go` | Types, the `Store` interface, `Service` and `New`. No `net/http`. |
| `handler.go` | `NewHandler` and `Routes()` |
| `events.go` | Event names and payloads |
| `errors.go` | Exported error sentinels, checked with `errors.Is` |
| `mem.go` | In-memory `Store` used by tests |
| `*_test.go` | Tests |

The Postgres implementation of each store lives in `adapters/postgres/`. Only `app/` imports adapters, so swapping a provider means swapping an adapter. `app/imports_test.go` enforces these import rules.

## The app package

`app/app.go` builds every component in `New` and mounts routes in `router`. `app/subscribe.go` registers event handlers, and `app/rbac.go` builds the `auth.Guard` from `org.Roles` and the permissions each role adds. Nothing is wired anywhere else.

## Events

Modules publish typed events on an in-process bus (`platform/events`) after a write succeeds, for example `projects.EventCreated`. Handlers in `app/subscribe.go` send emails, record analytics, and forward selected events to [outbound webhooks]({{< relref "webhooks" >}}).

An event in `projects/events.go`, published in `projects/service.go`:

```go
const EventCreated = "project.created"

type Created struct {
	ID    uuid.UUID `json:"id"`
	OrgID uuid.UUID `json:"org_id"`
	Name  string    `json:"name"`
}

events.Publish(s.bus, ctx, EventCreated, Created{ID: p.ID, OrgID: p.OrgID, Name: p.Name})
```

## Responses

Handlers use the `platform/httpx` writers:

- A single object: `{"object": ..., "data": ...}`
- A list: `{"object": "list", "data": [], "has_more": ..., "starting_after": ...}`
- An error: `{"object": "error", "error": {"type": ..., "message": ...}}`

Lists use cursor pagination with `starting_after` and `has_more`.

## Adding a module

Copy `projects/` as a starting point:

1. Create `service.go`, `mem.go` and `handler.go` (and `events.go` if the module publishes events).
2. Add a migration, the queries, and a Postgres store. See [Database & migrations]({{< relref "database" >}}).
3. In `app.New`, construct the store and service, then mount the routes in `router`.
4. For organization-scoped data, mount the routes with the `tenant` middleware and add the prefix to `app/isolation_test.go`.
5. Add the routes to `app/openapi.yaml`.

The sample module's routes, in `projects/handler.go`:

```go
func (h *Handler) Routes() http.Handler {
	r := chi.NewRouter()
	r.Get("/", h.List)
	r.Get("/{id}", h.Get)
	r.Group(func(r chi.Router) {
		r.Use(h.guard.RequirePerm(PermWrite))
		r.Post("/", h.Create)
		r.Patch("/{id}", h.Update)
		r.Delete("/{id}", h.Delete)
	})
	return r
}
```

And their mount in `router` in `app/app.go`:

```go
r.With(authed, tenant).Mount("/projects", projectHandler.Routes())
```

`AGENTS.md` in the repository has step-by-step recipes for modules, endpoints, events, jobs and migrations.
