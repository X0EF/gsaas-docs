---
title: Email
weight: -13
---

# Email

Email is sent over SMTP. Set `SMTP_HOST`, plus `SMTP_PORT`, `SMTP_TLS`, `SMTP_USER`, `SMTP_PASS` and `EMAIL_FROM` as needed. Any SMTP provider works. Without `SMTP_HOST`, emails are written to the log, which is handy for local development.

## Templates

Templates live in `platform/email/template.json`. Each has a `subject`, `text` and `html` body:

| Template | Sent when |
| --- | --- |
| `welcome` | A user registers |
| `magic_link` | A user asks for a sign-in link |
| `org_invite` | A user is invited to an organization |

## How emails are triggered

Modules don't send email directly. They publish events, and handlers in `app/subscribe.go` render a template and send it. To add an email, add a template and subscribe to the event that should trigger it.
