# Contributing

StoreSense is a personal project and a portfolio piece, not a product with a team.
Issues and pull requests are welcome, but it is built to one person's judgement and
that is deliberate.

## Before a change

1. Read [CONTEXT.md](CONTEXT.md) for the vocabulary and [AGENTS.md](AGENTS.md) for
   the invariants.
2. Skim [docs/adr/](docs/adr/). If your change contradicts an Accepted ADR, the pull
   request has to supersede that ADR — a new `docs/adr/NNNN-*.md` marking the old one
   `Superseded by NNNN` — not work around it silently.
3. Check [ROADMAP.md](ROADMAP.md), including the non-goals.

## Making the change

- Match the surrounding code — naming, comment density, structure. The existing
  modules are the style guide.
- Every model call goes through `api/app/llm.py`. Nothing else talks to a provider.
- Tests must pass offline: `make test` with nothing running.
- Lint both halves: `ruff check app tests` in `api/`, `npm run lint` in `web/`.
- The web build typechecks: `npm run build` in `web/`.

## Commits and pull requests

- Plain, understated, British English. Present tense — "Add the dead stock report".
- One concern per commit.
- The pull request body says what changed and why, and names any ADR it adds or
  supersedes.

## What a good pull request looks like

The commit history is the reference: a feature and its tests, or a fix and the test
that would have caught it, in a handful of small commits, each message describing one
thing.
