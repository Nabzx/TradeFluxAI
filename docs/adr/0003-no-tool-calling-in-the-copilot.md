# 0003. No tool-calling in the copilot; gather context up front

- **Status:** Accepted
- **Date:** 2026-09-05
- **Driven by:** "Add the copilot: live numbers, retrieval and SSE streaming" (2026-08-04)

## Context

The copilot answers two kinds of question: about the numbers ("how did we do this
week?") and about the store ("what's our returns policy?"). The obvious modern shape
is tool-calling — give the model a `get_metrics` tool and a `search_docs` tool and
let it decide what to fetch.

But the set of things worth looking up is small and known, and the models this is
built to run on are 8B-class local models that follow a multi-step tool protocol
unreliably.

## Decision

The copilot does not use tool-calling. On every question it builds a fresh metrics
snapshot and runs retrieval, wraps both in `<store_data>` and `<store_knowledge>`
tags inside the user turn, and asks once. See `api/app/chat.py`.

The context goes in the user turn, not a second system message: small models treat a
lone system message as instructions and a second one as text to read back, and an
early version had a 1B model answering by reciting the headings.

## Alternatives considered

- **Tool-calling** — one round trip slower, needs a model that can drive the
  protocol, and buys nothing when there are only two things to fetch and both are
  cheap.
- **Retrieval only, no metrics snapshot** — cannot answer "what do I need to
  reorder?", which is half of what the owner asks.

## Consequences

- The copilot works on a small local model, which is the point.
- Every answer pays for a metrics snapshot and an embedding search even when it only
  needed one. On this data both are milliseconds, so it does not matter.
- If the catalogue of lookupable things grows past a handful, this decision is worth
  revisiting — that is the trigger, not model fashion.
