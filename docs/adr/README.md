# Architecture Decision Records

One record per settled decision: the context, the choice, the alternatives, the
consequences. Nothing else.

- Copy [`0000-template.md`](0000-template.md) to `NNNN-short-title.md` with the next
  number.
- Keep it short. If it needs sections beyond the template, it is probably two
  decisions.
- A reversed decision is marked `Superseded by NNNN` in its status line, not deleted.

ADRs 0002 onward were written retroactively on 2026-09-05 — see
[ADR-0001](0001-record-architecture-decisions.md). Each records a decision already in
the code; the body names the commit or measurement behind it.

## Index

| # | Decision | Status |
| --- | --- | --- |
| [0001](0001-record-architecture-decisions.md) | Record architecture decisions | Accepted |
| [0002](0002-one-llm-gateway-provider-agnostic.md) | One LLM gateway, provider-agnostic, speaking the OpenAI shape | Accepted |
| [0003](0003-no-tool-calling-in-the-copilot.md) | No tool-calling in the copilot; gather context up front | Accepted |
| [0004](0004-errors-as-events-once-a-stream-has-started.md) | Once a stream has started, errors are events not status codes | Accepted |
| [0005](0005-forecast-ships-as-a-blend.md) | The stockout forecast ships as a 50/50 blend of a model and a moving average | Accepted |
| [0006](0006-closed-vocabularies-for-model-classification.md) | Model classification writes into a closed vocabulary | Accepted |
| [0007](0007-lora-fine-tune-evaluated-then-shipped-off.md) | The LoRA fine-tune is wired up, evaluated, and off by default | Accepted |
| [0008](0008-shared-password-auth-off-by-default.md) | One shared password, stateless tokens, off unless configured | Accepted |
| [0009](0009-synthetic-seeder-is-the-default-dataset.md) | A synthetic seeder is the default dataset, not a fixture | Accepted |
| [0010](0010-no-scheduler-inside-the-app.md) | No scheduler inside the app; the host schedules the weekly digest | Accepted |
