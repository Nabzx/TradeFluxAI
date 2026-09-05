# 0001. Record architecture decisions

- **Status:** Accepted
- **Date:** 2026-09-05
- **Driven by:** the README carries the reasoning for a dozen non-obvious choices in prose, where it is hard to cite and easy to lose

## Context

StoreSense is small but it is not shallow. A handful of decisions took real work to
get right — the forecast lost to a moving average before it drew with one, the
fine-tune learned the voice and kept inventing facts, the copilot deliberately has
no tool-calling. Today that reasoning lives in module docstrings and in long README
paragraphs. It is good writing, but it is not a record: there is no list of what was
decided, no date on it, and nothing a future change can be measured against.

## Decision

We record every settled decision as a short ADR in `docs/adr/`, numbered
sequentially, following [`0000-template.md`](0000-template.md). An ADR states the
context, the decision, the alternatives, and the consequences — nothing else.

ADRs 0002 onward are written retroactively on 2026-09-05. Each one records a decision
that already shipped; the body names the commit or the measurement that drove it. The
dates on those ADRs are the date they were written down, not the date the code
landed — the git history has the second one.

## Alternatives considered

- **Leave the reasoning in the README and docstrings** — it is already there, but it
  cannot be indexed, dated, or superseded, and it grows the README past the point
  anyone reads to the end.
- **One design doc** — goes stale as a whole the moment one section is wrong, and
  gives no natural place to record that a decision was later reversed.

## Consequences

- There is one place to learn why the system is shaped the way it is.
- A pull request that contradicts an Accepted ADR can be rejected by number, or the
  ADR superseded in the same PR.
- ADRs are cheap to write and cheap to supersede. A reversed decision is marked
  `Superseded by NNNN`, not deleted.
