# 0002. One LLM gateway, provider-agnostic, speaking the OpenAI shape

- **Status:** Accepted
- **Date:** 2026-09-05
- **Driven by:** "Add the LLM gateway" (2026-08-04); tested for real by the fine-tune (2026-08-17)

## Context

StoreSense calls a model for eight different things — the copilot, streaming chat,
embeddings, semantic search, vision tagging, sentiment, alert parsing, copy. If each
feature talked to a provider directly, then timeouts, retries, error types, token
counting and the retry-vs-don't-retry decision would be written eight times and drift
apart.

The project also has to run for free on a fresh clone with no account anywhere, and
still deploy against a paid provider without a code change.

## Decision

Every model call goes through `api/app/llm.py`. Nothing else in the codebase imports
`httpx` to talk to a provider. The gateway speaks the OpenAI chat-completions and
embeddings shape, so it works unchanged against Ollama, OpenAI, Together, Groq, vLLM,
or the LoRA adapter's own server. The default `base_url` points at a local Ollama.

The gateway owns:

- **Retry policy.** 5xx and 429 retry with exponential backoff; 4xx raises
  `LLMBadRequest` and stops, because a bad model name will not fix itself.
- **Error taxonomy.** `LLMUnavailable` (nothing reachable) vs `LLMBadRequest` (we
  sent something wrong) vs `LLMError` (base), so callers catch what they can handle.
- **Token accounting.** A completed call is handed to `on_usage`, set by the app at
  startup. The gateway never imports the database — a script or a test uses it
  untracked.

The copy generator takes its own `base_url` rather than switching the global one,
because the adapter serves chat only and everything else still needs a general model
and an embedder.

## Alternatives considered

- **A provider SDK per feature** — less code today, but locks the project to one
  vendor and scatters the retry logic.
- **A heavier abstraction (LiteLLM, LangChain)** — a dependency and a mental model
  larger than a 289-line file that does exactly what this project needs.

## Consequences

- Adding a provider is an environment variable, proven by `python -m app.check_provider`.
- The retry, timeout and error-handling tests (`tests/test_llm.py`) use a fake
  `httpx` transport and never touch the network, so they run in CI.
- The gateway is the one file where a subtle bug — retrying a 400, losing a token
  count, swallowing a stream error — would be expensive, so it carries the most
  comment-per-line in the codebase.
- Anything the OpenAI shape cannot express (Anthropic's native tool blocks, say) is
  not reachable without widening the gateway first.
