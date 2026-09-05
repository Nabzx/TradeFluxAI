# 0007. The LoRA fine-tune is wired up, evaluated, and off by default

- **Status:** Accepted
- **Date:** 2026-09-05
- **Driven by:** the fine-tune spike (`finetune/`), 2026-08-17

## Context

The copy generator writes product descriptions and win-back emails in noszn's voice —
lowercase, understated, no "elevate your wardrobe". A LoRA adapter on
Qwen2.5-0.5B-Instruct was trained on 51 hand-written examples to see whether a small
tuned model does this better than a prompted general one.

The voice transferred cleanly: average output fell from 52 words to 16, and
capitalised lines from 20 to 0 across the held-out set. But the 0.5B model invents
product facts — called a tee a hoodie, a ripstop cargo pant "ribbed cotton", made up
a "ready in 24 hours" delivery promise. For a feature whose whole job is describing
real garments, that is disqualifying: wrong fabric is a returns problem.

## Decision

Keep the adapter in the repo, fully wired, and **off by default**.
`COPY_LLM_BASE_URL` blank means the copy generator uses the main model, which is the
recommendation. The evaluation — including the null result on the banned-words
metric, kept in the table because a null result is a result — is written up in
`finetune/README.md`.

The adapter is served behind its own OpenAI-compatible endpoint and pointed at via
its own `base_url` (see [ADR-0002](0002-one-llm-gateway-provider-agnostic.md)), not a
global switch, because it serves chat only.

## Alternatives considered

- **Ship it on** — fails the one thing the feature has to do.
- **Delete it** — throws away a real, honest negative result and the proof that the
  gateway is provider-agnostic in fact and not just in the README.
- **Constrain the output to a template** — makes it shippable at 0.5B, but gives up
  most of the reason to fine-tune. Recorded in `finetune/README.md` as the option
  that exists if the feature is ever forced.

## Consequences

- A reader can turn it on in one environment variable and see the failure themselves.
- The path to making it work is written down and ordered by cost: bigger base first.
- Pointing the app at the adapter is the regression test for ADR-0002 —
  `check_provider` against a genuinely different model and prompt contract.
