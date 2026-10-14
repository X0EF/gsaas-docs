---
title: Testing
weight: -9
---

# Testing

`make check` is the definition of done. It runs:

1. `gofmt` (fails if any file needs formatting)
2. `go vet ./...`
3. `go test ./...`
4. `sqlc generate` into a temporary directory and a diff against `database/sqlc`, which fails if the generated code is stale. This step is skipped with a message if `sqlc` isn't installed.

`make test` runs only `go test ./...`. CI (`.github/workflows/check.yml`) runs `make check` on pull requests.

## What the tests cover

- Module tests run against the in-memory stores in each `mem.go`, so they need no database.
- Postgres store tests in `adapters/postgres` are skipped unless `DATABASE_URL` is set.
- `app/imports_test.go` enforces the package import rules.
- `app/isolation_test.go` checks that one organization can't reach another's data, and fails until every route under a tenant prefix is covered.
- `app/openapi_test.go` fails for any mounted route missing from `app/openapi.yaml`.
