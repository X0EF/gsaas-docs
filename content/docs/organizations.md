---
title: Organizations & roles
weight: -16
---

# Organizations & roles

Every user gets a personal organization they own, created at sign-up, first Supabase sign-in, or invite acceptance. Users can also create team organizations and invite members. Data in tenant modules such as projects, jobs, billing and webhooks belongs to one organization.

## Choosing the organization

Routes with `{orgID}` in the path use that organization. Other tenant routes, such as `/projects` and `/jobs`, read the `X-Org-ID` header. The tenant middleware checks membership, so a user who isn't a member gets 403. Isolation is enforced in app code and covered by `app/isolation_test.go`.

## Roles

There are four roles, in rank order: `viewer`, `member`, `admin`, `owner`.

## Permissions

Roles are ranked by `org.RoleRank`. For finer control, a module can define its own `auth.Perm` constants (for example `projects.PermWrite`). Permissions are optional: a role gate is often enough.

`app/rbac.go` builds the one `auth.Guard`: the role rank plus the policy mapping roles to permissions.

| Role | Permissions |
| --- | --- |
| owner | all permissions |
| admin | `org:update`, `org:billing`, `org:members:invite`, `org:api_keys`, project write, job read and write |
| member | job read |
| viewer | none |

`org:delete` is held only by owners.

## Gating routes

The guard gives two middlewares. `app/` passes the `*auth.Guard` to each handler that needs it.

```go
// That role or higher
r.With(guard.RequireRole(org.RoleAdmin)).Post("/", h.Create)

// A permission, resolved through the policy in app/rbac.go
r.With(h.guard.RequirePerm(projects.PermWrite)).Post("/", h.Create)
```

To add a permission, define an `auth.Perm` constant in your module and add it to roles in `app/rbac.go`.

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
