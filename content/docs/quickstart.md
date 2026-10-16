---
title: Quickstart
weight: -20
---

# Quickstart

Code: `Makefile`, `cmd/`, `docker-compose.yaml`

Run gsaas locally and make your first signed-in API call in about five minutes. Postgres runs in Docker; the API and worker run with Go.

## Prerequisites

- Go 1.26
- Docker with Compose
- `curl`
- sqlc v1.30.0, only if you change SQL queries

## Steps

1. **Create your settings.** In the repository root:

   ```bash
   cp .env.example .env
   ```

   The defaults work locally: Postgres on `localhost:5432`, the API on port `5000`.

2. **Start Postgres and add two users.**

   ```bash
   make seed
   ```

   This starts the Postgres container, runs migrations and adds `admin@local` and `member@local` (password `password`). Each gets a personal organization. The last log lines look like:

   ```
   level=INFO msg=analytics event=org.created props="map[org_id:... owner_id:... personal:true]"
   level=INFO msg=analytics event=user.registered props=map[user_id:...]
   ```

   Running it again is safe.

3. **Start the API.**

   ```bash
   make run
   ```

   Look for `msg=listen addr=[::]:5000`. Check it in another terminal:

   ```bash
   curl localhost:5000/healthz
   ```

   ```json
   {"data":{"ok":true,"version":"dev"},"object":"health"}
   ```

4. **Start the worker** in another terminal. It runs background jobs such as billing updates and webhook deliveries.

   ```bash
   go run ./cmd/worker
   ```

   It keeps running and logs nothing more until a job runs.

5. **Sign in.**

   ```bash
   curl -X POST localhost:5000/auth/signin \
     -H 'Content-Type: application/json' \
     -d '{"email":"admin@local","password":"password"}'
   ```

   ```json
   {"data":{"access_token":"eyJ...","expires_at":"...","refresh_token":"...","user_id":"..."},"object":"token"}
   ```

6. **Make your first authenticated call.** Send the `access_token` as a bearer token:

   ```bash
   curl localhost:5000/me -H 'Authorization: Bearer <access_token>'
   ```

   ```json
   {"data":{"id":"...","email":"admin@local","first_name":"Admin","last_name":"Local","is_platform_admin":true,...},"object":"user"}
   ```

7. **Explore the API.** Open `http://localhost:5000/docs` for the interactive API reference.

## Good to know

- Without `SMTP_HOST`, emails such as magic links are written to the server log.
- Change the seed users with `SEED_ADMIN_*` and `SEED_MEMBER_*` in `.env`.
- `make stop` stops the Postgres container. Data is kept in a Docker volume.

## Troubleshooting

| Problem | Fix |
| --- | --- |
| `DATABASE_URL is required` | `.env` is missing. Run the commands from the repository root after step 1. |
| Postgres won't start, or port `5432` is in use | Stop the other Postgres, or change the port in `docker-compose.yaml` and `DATABASE_URL`. |
| `address already in use` on start | Port `5000` is taken. Set `PORT` in `.env`, and use that port in the URLs above. |
| `{"error":{"message":"unauthorized",...}}` | The `Authorization` header is missing, or the access token expired. Sign in again. |
| Billing or webhooks never update | The worker isn't running (step 4). |

Next: [Configuration]({{< relref "configuration" >}}), [Billing]({{< relref "billing" >}}), [Webhooks]({{< relref "webhooks" >}}).
