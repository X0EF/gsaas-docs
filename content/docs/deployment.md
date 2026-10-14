---
title: Deployment
weight: -8
---

# Deployment

The repository has a multi-stage `Dockerfile`. It builds the API server as a static binary and runs it on a distroless, non-root image, listening on port 5000.

```bash
docker build --build-arg VERSION=1.0.0 -t gsaas .
```

`VERSION` is optional and is compiled into the binary.

The image runs the API server only. Your deployment also needs:

- Postgres, with `DATABASE_URL` pointing to it
- Migrations applied (`go run ./cmd/migrate up`) before starting a new version
- The background worker (`cmd/worker`) running as a separate process
- Production settings: your own `JWT_SECRET`, `APP_BASE_URL`, and any Stripe, SMTP and S3 variables you use. See [Configuration]({{< relref "configuration" >}}).

Never set `WEBHOOK_ALLOW_INSECURE` in production.

## Health checks

- `GET /healthz`: the process is up
- `GET /readyz`: the app can reach the database

## Local Docker

`docker-compose.yaml` runs Postgres. The API joins with the `app` profile:

```bash
make migrate
docker compose -f docker-compose.yaml --profile app up --build
```
