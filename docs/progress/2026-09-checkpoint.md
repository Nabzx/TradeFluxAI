# Checkpoint — September 2026

Where StoreSense stands after the first build and the decision-spine pass.

## Built and working

The vertical slice from August, all running against the synthetic noszn year:

- **Copilot** — SSE streaming, live metrics snapshot plus cited retrieval, voice
  input. No tool-calling ([ADR-0003](../adr/0003-no-tool-calling-in-the-copilot.md)).
- **Gateway** — one provider-agnostic path for every model call, retry taxonomy,
  token accounting ([ADR-0002](../adr/0002-one-llm-gateway-provider-agnostic.md)).
- **Stockout forecast** — pooled gradient-boosting model blended 50/50 with a
  28-day moving average, ~36.9% MAE vs the baseline's ~39.7% over four folds, with
  the comparison shown on the card
  ([ADR-0005](../adr/0005-forecast-ships-as-a-blend.md)).
- **Semantic search** over the catalogue; **review sentiment** into a closed theme
  vocabulary ([ADR-0006](../adr/0006-closed-vocabularies-for-model-classification.md));
  **plain-English alerts**; **vision tagging**; **weekly digest** with an
  out-of-process scheduler ([ADR-0010](../adr/0010-no-scheduler-inside-the-app.md)).
- **LoRA fine-tune** — evaluated, learned the voice, invents facts at 0.5B, ships
  off ([ADR-0007](../adr/0007-lora-fine-tune-evaluated-then-shipped-off.md)).
- **Auth** — shared password, stateless tokens, off unless configured
  ([ADR-0008](../adr/0008-shared-password-auth-off-by-default.md)).
- **CI** — lint and offline tests on both halves. Suites: gateway, retrieval, auth,
  usage, digest, digest Lambda, Shopify sync, dead stock, copywriter.

## Added in this pass

- `docs/adr/` — ten records covering the decisions that were previously only in
  README prose and docstrings.
- `CONTEXT.md` — the domain vocabulary.
- `AGENTS.md` — the invariants, each linked to the ADR behind it.
- `ROADMAP.md` — four provisional phases and an explicit non-goals list.
- `CONTRIBUTING.md`.

## Not done

- No live deployment. `DEPLOY.md` has the steps; nobody has run them.
- No demo video, no screenshots beyond the one in the README.
- Never connected to the real noszn store — the Shopify API gaps in the README are
  documented from the docs, not from a sync that happened.
- The forecast and retrieval evaluation live in code and prose, not in a `bench/`
  harness with a CI regression gate.
- Single-tenant throughout.

## Next

Phase 1 in `ROADMAP.md`: deploy the demo, record it, run the real store through it.
