---
title: Configuration
weight: -19
---

# Configuration

Code: `platform/config/config.go`, `.env.example`

All settings come from environment variables, read in `platform/config/config.go`. Copy `.env.example` to `.env` to start. Every setting has a default, so the server boots with only `DATABASE_URL` set.

| Area | Variables |
| --- | --- |
| Server | `APP_ENV`, `PORT`, `SHUTDOWN_TIMEOUT`, `MAX_BODY_BYTES`, `CORS_ALLOWED_ORIGINS`, `AUTH_RATE_LIMIT`, `APP_BASE_URL`, `APP_NAME` |
| Database | `DATABASE_URL` (required) |
| Local auth | `JWT_SECRET`, `ACCESS_TTL`, `REFRESH_TTL`, `MAGIC_LINK_TTL`, `INVITE_TTL` |
| Supabase auth | `SUPABASE_URL` |
| Billing | `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`, `STRIPE_PRICE_PRO`, `STRIPE_PRICE_ITEM` |
| Billing redirects | `FRONTEND_BASE_URL`, `CHECKOUT_SUCCESS_PATH`, `CHECKOUT_CANCEL_PATH`, `PORTAL_RETURN_PATH` |
| Email | `SMTP_HOST`, `SMTP_PORT`, `SMTP_TLS`, `SMTP_USER`, `SMTP_PASS`, `EMAIL_FROM` |
| File storage | `S3_ENDPOINT`, `S3_REGION`, `S3_PUBLIC_URL`, `S3_ACCESS_KEY_ID`, `S3_SECRET_ACCESS_KEY`, `S3_PUBLIC_BUCKET`, `S3_PRIVATE_BUCKET`, `FILE_EXPIRES`, `UPLOAD_MAX_BYTES` (default 10 MiB) |
| Outbound webhooks | `WEBHOOK_ALLOW_INSECURE` (local development only) |
| Seed users | `SEED_ADMIN_*`, `SEED_MEMBER_*` (`EMAIL`, `PASSWORD`, `FIRST_NAME`, `LAST_NAME`) |

Optional integrations turn on when you set their variables:

- With no `STRIPE_SECRET_KEY`, the app runs without a billing processor.
- With no `SMTP_HOST`, emails are written to the log rather than sent.
- With no `S3_ACCESS_KEY_ID` or `S3_ENDPOINT`, no file storage is configured.

Always set your own `JWT_SECRET` outside local development. The built-in default is for local use only.

To add a setting, add a field with a default to `Config` in `platform/config/config.go`.
