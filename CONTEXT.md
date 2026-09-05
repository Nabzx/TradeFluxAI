# StoreSense

An AI dashboard for one Shopify clothing brand. The owner opens one page, sees how
the shop is doing, asks it questions in plain English, and finds out a size is going
before it goes. It runs with no credentials and no paid keys against a synthetic
year of data, and against a real store when pointed at one.

This file fixes the vocabulary. When code, comments, commits and docs use the same
words for the same things, there is less to hold in your head. Where a term has a
tempting near-synonym, it is listed under _Avoid_.

## The product

**StoreSense**
The whole thing: the FastAPI backend, the Next.js dashboard, and the model calls
behind them. Not the name of any one component.

**Owner**
The single person the dashboard is for. There are no roles and no user table — see
[ADR-0008](docs/adr/0008-shared-password-auth-off-by-default.md).
_Avoid_: user, admin, tenant

**Copilot**
The assistant docked on the right of the dashboard. Answers questions about the
numbers and about the store's own documents, streaming the reply. Lives in
`api/app/chat.py`.
_Avoid_: chatbot, agent, assistant (unqualified)

## The copilot's inputs

**Snapshot**
The compact, freshly-built picture of how the store is doing right now — last 7 and
30 days, top products, what is sold out, what is running out — that goes in front of
every copilot question. Built by `chat.build_snapshot`.
_Avoid_: context (unqualified), summary, dump

**Store knowledge**
The handful of document chunks retrieval pulls for a question, wrapped in
`<store_knowledge>` tags in the prompt. Comes from the knowledge folder and the
catalogue.
_Avoid_: context (unqualified), docs, RAG results

**Citation**
A bracketed number — `[1]` — in a copilot answer that matches a numbered chunk in
the store-knowledge block. The copilot cites knowledge, never the snapshot.
_Avoid_: reference, source link, footnote

**Knowledge folder**
`api/app/knowledge/` — the hand-written shipping, returns and sizing documents the
copilot cites. Not synced from Shopify; replaced by hand for a real store.
_Avoid_: docs folder, corpus

## Retrieval

**Chunk**
One unit of retrievable text with the file and heading it came from. A markdown
section, or one product written as a sentence. Stored in the `Chunk` table with its
embedding.
_Avoid_: passage, segment, node

**Embedder**
Whatever turns text into vectors. `ModelEmbedder` calls the gateway;
`TfidfEmbedder` is the no-model fallback that has to see the corpus before it can
embed. The query embedder must match the one that built the index.
_Avoid_: encoder, vectoriser, embedding model (when the fallback is meant too)

**Index**
The current set of embedded chunks. Derived data — dropped and rebuilt on
`python -m app.rag` or after a Shopify sync, never migrated.
_Avoid_: vector store (that is the module), database

## The catalogue

**Catalogue**
The products and their variants. One of the two things retrieval covers.
_Avoid_: inventory (that is the stock numbers), shop

**Variant**
One size of one product — "hoodie / M". The unit the forecast predicts and the
stockout report is keyed on.
_Avoid_: SKU (that is the code), size (informal only), item

**Active / archived**
A product's status. Archived lines still train the forecast and still count toward
past revenue, but are kept out of stock reports and search — a discontinued line is
not a buying decision.
_Avoid_: deleted, hidden, discontinued (use archived)

## Forecasting

**Stockout forecast**
The prediction of how many days of stock each in-stock variant has left, soonest
first. `forecast.stockout_report`.
_Avoid_: demand forecast (that is the intermediate step), prediction

**Baseline**
The 28-day moving average of a variant's daily sales — what the owner would work out
by eye. Everything the forecast does is measured against it.
_Avoid_: naive model, benchmark, control

**Blend**
What actually ships: half the model's rate, half the baseline's. Not a hedge — it
measured better than either alone. See
[ADR-0005](docs/adr/0005-forecast-ships-as-a-blend.md).
_Avoid_: ensemble, average (unqualified)

**Backtest**
The rolling four-fold check that scores the blend, the model alone and the baseline
alone on total units per variant. Its numbers are shown under the forecast card.
_Avoid_: validation, evaluation (unqualified), CV

**Error floor**
The mean absolute error no forecast can beat on this data because the sales are
Poisson noise — 32.6% across the catalogue. Both the model and the baseline sit on
it.
_Avoid_: irreducible error, noise (unqualified)

## Reviews

**Theme**
The single thing a review is most about, chosen from a closed list of nine. Free-text
themes do not add up — see
[ADR-0006](docs/adr/0006-closed-vocabularies-for-model-classification.md).
_Avoid_: topic, tag, category, aspect

**Insight**
The output of the sentiment card: the positive/neutral/negative split, the themes
behind each, and a few real negative reviews to read. `sentiment.insights`.
_Avoid_: analysis, report

## Alerts

**Alert**
A rule the owner created by typing a sentence — "warn me when a hoodie has less than
two weeks of stock left". Stored with both the original phrase and the parsed rule.
_Avoid_: notification, trigger, watch

**Reads-as**
The plain-English description of how a phrase was parsed, shown back to the owner so
they can confirm the rule was understood. `alerts.describe`.
_Avoid_: explanation, preview

## Delivery

**Digest**
The short weekly brief — last week plus what to do about it. Readable on the
dashboard, printable from the CLI, emailed on a schedule the app does not own. Also
"the Monday brief".
_Avoid_: newsletter, summary email, report

## Model access

**Gateway**
`api/app/llm.py`. The one place any model call happens — chat, streaming, embeddings,
vision. Provider-agnostic, speaks the OpenAI shape. See
[ADR-0002](docs/adr/0002-one-llm-gateway-provider-agnostic.md).
_Avoid_: client (unqualified), LLM wrapper, provider

**Provider**
Whatever the gateway is pointed at — Ollama by default, or OpenAI, Groq, vLLM, the
LoRA adapter's server. Selected by environment variable, never in code.
_Avoid_: backend, vendor, API

**Usage**
The token count and pound cost recorded for every completed model call, tagged by
which feature spent it. `api/app/usage.py`, shown on the dashboard.
_Avoid_: telemetry, metrics (those are the store's numbers), analytics

## Data

**Seeder**
`python -m app.seed`. Generates the synthetic noszn year — a real dataset with
structure, not fixtures. See
[ADR-0009](docs/adr/0009-synthetic-seeder-is-the-default-dataset.md).
_Avoid_: fixtures, mock data, sample data

**Data source**
Whether the loaded data is `demo` (the seeder) or `shopify` (a real sync). Recorded
by `datasource.py` and shown in the dashboard header.
_Avoid_: mode, environment
