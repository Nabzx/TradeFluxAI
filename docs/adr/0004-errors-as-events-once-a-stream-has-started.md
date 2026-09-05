# 0004. Once a stream has started, errors are events not status codes

- **Status:** Accepted
- **Date:** 2026-09-05
- **Driven by:** "Add the copilot: live numbers, retrieval and SSE streaming" (2026-08-04); frame parsing pulled into one place (2026-08-05)

## Context

The copilot and the copy generator stream their answers as server-sent events. Once
the first token has been sent, the HTTP response is already `200` and the headers are
gone. If the model then fails halfway through a sentence, there is no status code
left to return — the browser has started reading a `200 OK`.

## Decision

The gateway retries only the opening of a stream. After the first token, a failure
is raised, not retried — the user has already read half a sentence and cannot have it
silently restarted.

The route layer catches that failure and sends it as an SSE event:
`{"type": "error", "message": "..."}`, followed by `data: [DONE]`. The dashboard
renders whatever text arrived plus the warning. Context-gathering failures before the
stream opens are sent the same way, because the response has already been handed to
`StreamingResponse` by then.

`X-Accel-Buffering: no` is set on every streaming response so nginx does not buffer
the stream and defeat the point.

## Alternatives considered

- **Retry mid-stream** — produces a garbled answer with a repeated or contradictory
  middle.
- **Drop the connection on error** — the browser cannot tell a crash from a clean
  finish, so the user sees a truncated answer with no explanation.
- **Buffer the whole answer, then send** — gives back the status code, but throws
  away the reason the copilot streams at all.

## Consequences

- The frontend has one error path for streamed features: read `type: "error"` events.
- Retry coverage on streamed calls is genuinely thinner than on `complete()` — this
  is a known, accepted gap, not an oversight.
- Any new streamed endpoint must go through `sse_stream()` / `chat.sse()` so it
  inherits this shape.
