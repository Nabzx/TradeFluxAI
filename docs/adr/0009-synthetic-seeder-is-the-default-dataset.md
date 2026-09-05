# 0009. A synthetic seeder is the default dataset, not a fixture

- **Status:** Accepted
- **Date:** 2026-09-05
- **Driven by:** "Add synthetic noszn seeder: a year of orders, stock and reviews" (2026-08-04)

## Context

Every feature in StoreSense needs data with real structure: the forecast needs a
year of daily sales with seasonality, the sentiment card needs reviews that cluster
on a theme, the metrics need period-over-period movement. A handful of fixture rows
gives none of that, and pointing the project at a real Shopify store to try it out
needs credentials, `read_all_orders` approval, and a store.

## Decision

`python -m app.seed` generates a synthetic year for noszn day by day — quiet Mondays,
busy weekends, hoodies picking up in autumn, stock draining between restocks — so the
patterns are real enough for the forecaster to find. It is the default dataset: a
fresh clone seeds and everything works, with no credentials and no paid keys.

`datasource.py` records whether the loaded data is `demo` or a real Shopify sync, and
the dashboard header says which. A real sync replaces the synthetic data wholesale.

No real customer data is in the repo, and the Shopify sync anonymises names and
emails on the way in.

## Alternatives considered

- **Static fixtures** — cannot exercise the forecast or the trend metrics, which are
  most of the product.
- **Require a real store to demo** — makes the project unrunnable for anyone
  evaluating it, which defeats the point of open-sourcing it.
- **Record and replay a real store** — a privacy problem and a licensing question
  for data that is not the author's to publish.

## Consequences

- The project is runnable end to end by anyone, immediately — the main thing a
  reviewer or a hiring manager will actually do.
- The seeder is real code with real bugs available to it; a distribution shift
  between synthetic and real data is a risk the forecast backtest would surface.
- Numbers in screenshots and the demo are synthetic, and are labelled as such by the
  header rather than in a disclaimer nobody reads.
