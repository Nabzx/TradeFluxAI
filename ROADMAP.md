# Roadmap

Where StoreSense is and where it is going. Provisional — this is the current
thinking, not a commitment. Each phase should end with something runnable and a
progress note in [docs/progress/](docs/progress/).

## Where it is now

A working vertical slice, built August 2026. Everything in the README runs against a
synthetic year of data: the copilot, cited retrieval, the stockout forecast with its
backtest, semantic search, plain-English alerts, review sentiment, vision tagging,
voice, the weekly digest, token and cost tracking, and a shared-password login.
Tests and CI cover the gateway, retrieval, auth, the digest and the Lambda.

Not done: no live deployment, no demo video, no real store connected, and the
decision records this file sits next to were only written in September.

## Phase 1 — Make it real

The project has never run against anything but its own seeder.

- [ ] Deploy the demo — API on Render, dashboard on Vercel, Postgres on Neon, seeded
      synthetic store, public URL. `DEPLOY.md` already has the steps.
- [ ] Record the 90-second demo: a real question to the copilot, the forecast card,
      an alert created from a sentence.
- [ ] Point it at the real noszn store for two to three weeks. Write up what broke —
      the Shopify API gaps are already listed in the README, this is where they get
      tested.
- [ ] Surface the cost tracking that already exists as a visible `/health` +
      spend-per-day view.

**Done when:** a stranger can open a URL and use it, and one real store has run
through it.

## Phase 2 — Depth on the measurement

The strongest work in the repo — the forecast that was measured, not assumed — is
currently a paragraph in the README. Make it a harness.

- [ ] `bench/forecast/` — the rolling backtest as a committed, reproducible report:
      the blend, the model alone, the baseline, the noise floor, over more folds
      than the dashboard runs.
- [ ] `bench/rag/` — a 30-to-50 question evaluation set over the knowledge folder,
      scoring retrieval precision@k **and** whether the copilot cited the chunk it
      actually used.
- [ ] A CI job that runs both and fails on a regression past a threshold.
- [ ] A feedback capture — "the copilot got this wrong" — that grows the RAG
      evaluation set from real use.

**Done when:** no AI feature ships a change without a number moving in `bench/`.

## Phase 3 — More than one shop

Every part of StoreSense assumes one store: one password, one database, one
`store_name`, one forecast cache. Multi-tenancy is the change that turns it from a
dashboard into a product.

- [ ] A store identifier threaded through the models, the sync, and the cache.
- [ ] Per-store credentials and per-store isolation of data and usage.
- [ ] Auth that models more than one person — the first thing
      [ADR-0008](docs/adr/0008-shared-password-auth-off-by-default.md) says has to
      change.

This is a real rewrite of several seams, not a config flag. It waits until Phase 1
has found a second store that wants it.

## Phase 4 — Harden the model layer

The gateway is the most-reused code in the project. Give it the depth its position
deserves.

- [ ] Contract tests against a fake provider; property tests on the backoff.
- [ ] Prompt-injection tests on the retrieval path — what happens when a knowledge
      document contains "ignore previous instructions".
- [ ] Structured-output validation at the boundary for tagging, alerts and sentiment,
      extending the closed-vocabulary rule from
      [ADR-0006](docs/adr/0006-closed-vocabularies-for-model-classification.md).

## Non-goals

Things StoreSense is deliberately not, so a contribution proposing one gets a fast
answer:

- **Not a Shopify app-store app.** It is a self-hosted dashboard for a store you
  own, not a listing with an OAuth install flow and Shopify's review process.
- **Not multi-channel.** Shopify only. No Amazon, Etsy, or a generic connector
  framework.
- **Not a BI tool.** The metrics exist to feed the copilot and the alerts and to
  answer the owner's actual questions, not to be a pivot-table builder.
- **Not agentic.** The copilot gathers a known, small set of context and answers
  once — see [ADR-0003](docs/adr/0003-no-tool-calling-in-the-copilot.md). No tool
  loops, no autonomous actions against the store.
- **Not a fine-tuning project.** The LoRA adapter is a measured experiment that
  ships off ([ADR-0007](docs/adr/0007-lora-fine-tune-evaluated-then-shipped-off.md)).
  The product runs on prompted general models.
- **Not built for scale.** In-memory rate limiting, a training run in the web
  process, a forecast cache in a module global. Every one of these is fine for one
  shop and called out in the code where a bigger deployment would need to change it.
