---
title: Quickstart
weight: -20
---

# Quickstart

Code: `Makefile`, `cmd/`, `docker-compose.yaml`

Requirements: Go 1.26, Docker, and sqlc v1.30.0 (only to regenerate queries).

```bash
cp .env.example .env
make seed              # starts Postgres, runs migrations, adds two local users
make run               # API on localhost:5000
go run ./cmd/worker    # background jobs, in another terminal
```

Or run everything in Docker:

```bash
cp .env.example .env
make migrate
docker compose -f docker-compose.yaml --profile app up --build
```

Then open `http://localhost:5000/docs` for the interactive API reference.

`make seed` adds `admin@local` and `member@local` (password `password`); change them with `SEED_ADMIN_*` and `SEED_MEMBER_*`. Without `SMTP_HOST`, emails such as magic links are written to the server log.

Next: [Configuration]({{< relref "configuration" >}}), [Billing]({{< relref "billing" >}}), [Webhooks]({{< relref "webhooks" >}}).
