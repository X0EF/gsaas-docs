---
title: Authentication
weight: -17
---

# Authentication

Clients send `Authorization: Bearer <token>` on each request. The token is either a session token or an organization API key. Use one auth mode: local (the default) or Supabase.

## Local auth (default)

The `identity` module issues and verifies session JWTs signed with `JWT_SECRET`. Access tokens last `ACCESS_TTL` and refresh tokens last `REFRESH_TTL`.

| Route | Purpose |
| --- | --- |
| `POST /auth/signup` | Create an account |
| `POST /auth/magic-link` | Email a one-time sign-in link |
| `POST /auth/magic-link/verify` | Exchange the link token for a session |
| `POST /auth/signin` | Sign in with email and password |
| `POST /auth/refresh` | Get a new access token |
| `POST /auth/signout` | End the session |
| `PUT /me/password` | Set an optional password |

Magic links are the default sign-in and expire after `MAGIC_LINK_TTL`. `/auth` routes are rate limited (`AUTH_RATE_LIMIT`).

## Supabase auth

To use Supabase Auth instead, swap the token verifier in the commented block in `app.New` (`app/app.go`), set `SUPABASE_URL`, and remove the `/auth` mount in `router`. `adapters/supabase` verifies access tokens locally through JWKS (ES256 or RS256) and checks `iss`, `aud` and `exp`. On a user's first request, a matching user row is created with the same ID as the Supabase `sub`.

Invite acceptance (`POST /invites/accept`) stays local in both modes.

## API keys

API keys belong to an organization and act for it with a fixed role and no user. Tokens starting with `ak_` are checked as API keys; every other token is checked as a session.

| Route | Purpose |
| --- | --- |
| `POST /organizations/{orgID}/api-keys` | Create a key |
| `GET /organizations/{orgID}/api-keys` | List keys |
| `DELETE /organizations/{orgID}/api-keys/{keyID}` | Revoke a key |

A key can only reach its own organization. Some routes need a signed-in user and return 403 for API keys: `/me`, `/users`, `/files`, `POST /organizations`, and the organization invites, API keys and billing routes.

## Users

| Route | Purpose |
| --- | --- |
| `GET`, `PATCH`, `DELETE /me` | Read, update or delete your account |
| `GET /users` | List users (platform admins only) |
| `GET`, `PATCH /users/{id}` | Read or update a user |
