---
title: Organizations & multi-tenancy
weight: -16
---

# Organizations & multi-tenancy

Code: `organization/`, `platform/auth/`, `app/rbac.go`

Every user gets a personal organization they own, created at sign-up, first Supabase sign-in, or invite acceptance. Users can also create team organizations and invite members. Data in tenant modules such as projects, jobs, billing and webhooks belongs to one organization.

## Choosing the organization

Routes with `{orgID}` in the path use that organization. Other tenant routes, such as `/projects` and `/jobs`, read the `X-Org-ID` header. The tenant middleware checks membership, so a user who isn't a member gets 403. Isolation is enforced in app code and covered by `app/isolation_test.go`.

## Roles

There are four roles. `org.Roles` lists them lowest first: `viewer`, `member`, `admin`, `owner`.

## Permissions

`app/rbac.go` builds the one guard with `auth.NewGuard(org.Roles, grants)`. `grants` lists the permissions each role adds, and each role inherits every permission of the roles below it. Permissions are optional `auth.Perm` constants that each module defines (for example `projects.PermWrite`).

| Role | Adds |
| --- | --- |
| viewer | nothing |
| member | job read |
| admin | `org:update`, `org:billing`, `org:members:invite`, `org:api_keys`, `org:webhooks`, project write, job write |
| owner | all permissions |

Only owners have `org:delete`, through all permissions.

## Gating routes

The `tenant` middleware resolves the org (`{orgID}` path, else `X-Org-ID`), checks the caller is a member and sets their role. An API key belongs to one org and keeps its own role.

The guard gives two middlewares. Use `RequireRole` for most routes; reach for `RequirePerm` only when you need finer control than the role ladder. In `app.go` you use `guard` directly. Inside a module, `NewHandler(s, guard)` stores it on the handler as `h.guard`.

```go
// app.go: that role or higher
r.With(guard.RequireRole(org.RoleAdmin)).Post("/", h.Create)

// Module handler: a permission granted to the role (or a role below it) in app/rbac.go
r.With(h.guard.RequirePerm(projects.PermWrite)).Post("/", h.Create)
```

To add a permission, define an `auth.Perm` constant in your module and add it to the lowest role that should have it in `app/rbac.go`.

```go
// projects/handler.go
const PermWrite auth.Perm = "projects:write"

// app/rbac.go
org.RoleAdmin: {org.PermOrgUpdate, org.PermOrgBilling, org.PermOrgMembersInvite, org.PermOrgAPIKeys, org.PermOrgWebhooks, projects.PermWrite, jobs.PermWrite},
```

## Organization routes

| Route | Purpose |
| --- | --- |
| `POST /organizations` | Create an organization |
| `GET /organizations/{orgID}` | Read it |
| `PATCH /organizations/{orgID}` | Update it (`org:update`) |
| `DELETE /organizations/{orgID}` | Delete it (owners only) |

A personal organization takes no invites and can't be deleted through this route. It is deleted with its owner's account.

## Invites

| Route | Purpose |
| --- | --- |
| `POST /organizations/{orgID}/invites` | Invite by email with a role |
| `GET /organizations/{orgID}/invites` | List pending invites |
| `DELETE /organizations/{orgID}/invites/{inviteID}` | Revoke an invite |
| `POST /invites/accept` | Accept an invite |

Invites need `org:members:invite`, can't grant the owner role, and expire after `INVITE_TTL`. The invitee gets an email with the invite link.
