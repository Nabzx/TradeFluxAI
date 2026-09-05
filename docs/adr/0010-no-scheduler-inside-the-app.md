# 0010. No scheduler inside the app; the host schedules the weekly digest

- **Status:** Accepted
- **Date:** 2026-09-05
- **Driven by:** "Add the Monday morning digest" (2026-08-10); "Send the weekly digest from an AWS Lambda" (2026-08-14)

## Context

The weekly digest is the one piece of StoreSense that comes to the owner rather than
waiting to be opened — a short brief, emailed on Monday morning. Something has to
trigger it on a timer.

A web process quietly running a background thread that emails on a schedule is a
surprising thing to find in a service: it fires only if the process happens to be up,
it double-fires if two instances run, and it is invisible until it breaks.

## Decision

There is no scheduler in the app. `python -m app.digest` prints the brief;
`--send` emails it. Triggering that on a cadence is the host's job, and every host
already has a mechanism.

The reference trigger is AWS EventBridge waking a Lambda (`aws/digest_scheduler/`).
The Lambda is a single file with no dependencies — it signs in to the API and calls
`POST /api/digest/send`. `DEPLOY.md` also documents a Render cron job and a plain
crontab as equal alternatives.

## Alternatives considered

- **APScheduler / a background task in the FastAPI process** — ties a scheduled job's
  reliability to the web process's uptime and instance count.
- **A Celery beat worker** — a broker, a worker process and a dependency, to send
  four emails a month.

## Consequences

- The app has no cron state to reason about; the digest is a pure function of the
  data plus a CLI flag.
- The scheduling story is per-host and lives in `DEPLOY.md` and `aws/README.md`, not
  in the code.
- The Lambda authenticates with the same shared password as everything else
  ([ADR-0008](0008-shared-password-auth-off-by-default.md)), sitting in an
  environment variable — fine for one shop, Secrets Manager if it ever holds more.
