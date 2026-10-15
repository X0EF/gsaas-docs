---
title: Database & migrations
weight: -10
---

# Database & migrations

Code: `database/` (migrations, queries, generated sqlc), `adapters/postgres/`, `cmd/migrate/`

gsaas uses Postgres. Migrations run with goose and queries compile to Go with sqlc. Local Postgres runs in Docker Compose.

## Make targets

| Target | What it does |
| --- | --- |
| `make postgres` | Start the Postgres container |
| `make migrate` | Run pending migrations |
| `make migrate-down` | Roll back the last migration |
| `make migrate-down-all` | Roll back all migrations |
| `make migrate-version` | Show the current version and status (read-only) |
| `make seed` | Migrate and add local admin and member users |
| `make sqlc-gen` | Regenerate Go code from SQL queries |

## Layout

- `database/postgres/migrations/`: numbered goose migrations, for example `00003_projects.sql`
- `database/postgres/query/`: SQL queries with `-- name:` annotations
- `database/sqlc/`: generated code. Never edit it by hand.
- `adapters/postgres/`: stores that implement each module's `Store` interface

## Adding a table and queries

1. Create `database/postgres/migrations/0000N_<name>.sql` with the next number, including `-- +goose Up` and `-- +goose Down`.
2. Add queries to `database/postgres/query/<module>.sql`.
3. Run `make sqlc-gen`.
4. Implement the store method in `adapters/postgres/<module>.go`, and the same method in the module's `mem.go`.
