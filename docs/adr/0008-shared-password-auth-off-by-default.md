# 0008. One shared password, stateless tokens, off unless configured

- **Status:** Accepted
- **Date:** 2026-09-05
- **Driven by:** "Add a shared password for the dashboard" (2026-08-07)

## Context

StoreSense is one shop, one owner. There are no roles to model and no user table
worth having. What the auth has to stop is a stranger who finds the deployed URL
running up an LLM bill — the expensive endpoints are the AI ones.

It also has to not get in the way of a fresh clone, which must run straight after
seeding with zero setup.

## Decision

A single shared password, set via `APP_PASSWORD`. Logging in swaps it for a token
that carries its own expiry and an HMAC signature, so nothing is stored server-side
and a restart does not log anyone out (given `SESSION_SECRET`).

**Auth is off entirely unless `APP_PASSWORD` is set.** A fresh clone is open, which
is what you want on a laptop. Because "I forgot to set it" is the likely failure, the
API prints a warning at startup when it is missing and `/health` reports
`auth_required`.

Enforcement is one HTTP middleware in front of every route, not a dependency on each
route — impossible to forget on a route added later. Password and signature
comparisons use `hmac.compare_digest`. The login endpoint has its own tighter rate
limit (5/min) because its requests are guesses. CORS middleware is registered last so
a `401` still carries the headers the browser needs to read it (see the comment in
`main.py`).

## Alternatives considered

- **A real user table with hashed passwords** — models a multi-user problem this
  product does not have.
- **OAuth / a provider login** — a dependency and a setup step for a two-person
  dashboard.
- **On by default with a generated password** — breaks "clone and run" and pushes a
  secret into the first-run output.

## Consequences

- `tests/test_auth.py` covers token signing, expiry and the constant-time compare
  without a running server.
- A deployment with no password is genuinely open. The warning and the `/health`
  flag are the mitigation; `DEPLOY.md` says it in bold.
- If StoreSense ever holds more than one shop's data, this is the first thing that
  has to change — it is called out in `ROADMAP.md` under multi-tenancy.
