# Working on StoreSense

For humans and agents making changes. Read [CONTEXT.md](CONTEXT.md) for the
vocabulary and [docs/adr/](docs/adr/) for why the system is shaped the way it is.

## Where things live

- **Decisions** — `docs/adr/`. A change that contradicts an Accepted ADR needs to
  supersede it in the same pull request, not ignore it.
- **Vocabulary** — `CONTEXT.md`. Use those words in code, comments and commits.
- **Progress checkpoints** — `docs/progress/`. Where the project stands, updated at
  milestones rather than continuously.
- **Roadmap** — `ROADMAP.md`, including what StoreSense deliberately will not do.

## Invariants

These hold across the codebase. Breaking one is a bug, not a judgement call.

### Every model call goes through the gateway

`api/app/llm.py` is the only place that opens an HTTP connection to a model
provider. No other module imports `httpx` (or a provider SDK) to call a model. This
is what keeps retries, timeouts, the error taxonomy and token accounting written
once. See [ADR-0002](docs/adr/0002-one-llm-gateway-provider-agnostic.md).

### Tests never need a model or the network

The gateway tests use a fake `httpx` transport; retrieval tests use a fake embedder.
`python -m pytest` passes offline with nothing running, and CI runs it that way. A
test that needs Ollama up does not belong in the suite.

### A fresh clone runs with no credentials

Every setting in `api/app/config.py` has a default that works. `make demo` then
`make api` and `make web` must produce a working dashboard with no `.env`, no keys,
no Shopify store. Paid providers and real stores are opt-in, never required.

### Auth is one gate, and off unless configured

The login check is a single HTTP middleware in `main.py` in front of every route —
not a per-route dependency that a new route can forget. It is inert unless
`APP_PASSWORD` is set. CORS middleware is registered **last** so a `401` still
reaches the browser. See [ADR-0008](docs/adr/0008-shared-password-auth-off-by-default.md).

### Streamed failures are events, not exceptions past the response start

Once a streaming response has yielded its first token, an error is sent as
`{"type": "error", ...}` followed by `data: [DONE]`. Nothing raises past that point.
New streamed endpoints go through `sse_stream()` / `chat.sse()`. See
[ADR-0004](docs/adr/0004-errors-as-events-once-a-stream-has-started.md).

### Model classification is validated before it is stored

Anything a model returns that is meant to be a label — a review theme, a sentiment,
an alert field — is checked against its allowed set and dropped if it does not
match. A dropped row is retried next run; a bad row is never written. See
[ADR-0006](docs/adr/0006-closed-vocabularies-for-model-classification.md).

### The query embedder matches the index embedder

Retrieval embeds the query with the same embedder that built the stored index
(`rag.get_query_embedder` reads it back from the index). Mixing them gives results
that look ranked and are random.

### The forecast shows its own accuracy

The forecast card keeps rendering the backtest comparison — blend, model alone,
baseline alone. A forecast on a shop this small that hides its error rate is
asserting, not predicting. See [ADR-0005](docs/adr/0005-forecast-ships-as-a-blend.md).

### No customer data in the repo

The seeder is synthetic. The Shopify sync anonymises names and emails on the way in.
Nothing that identifies a real customer is committed or stored as it arrives.

## Style

- Match the surrounding code. Comments explain *why*, and the existing modules set
  the density — a tricky decision gets a paragraph, a plain function gets nothing.
- Commits: plain, understated, British English, present tense — "Add the dead stock
  report", not "Added amazing new dead-stock analytics!". One concern per commit.
- Keep the README honest. If a feature has a known failure, the README says so — see
  the forecast and fine-tune sections.
