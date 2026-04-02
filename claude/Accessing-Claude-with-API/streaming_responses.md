# Claude API — streaming responses

Why **streaming** exists, how Anthropic exposes **SSE-style events**, and how to show text **incrementally** while still obtaining a **full assistant message** for history and storage.

## Overview

A non-streaming `messages.create` call returns only after the model finishes generating. For many models and prompts that can mean **roughly tens of seconds** (sometimes more), depending on model, prompt size, and `max_tokens`. Users perceive long waits better when they see **partial output** as it arrives—streaming solves that UX problem.

Technically, streaming means the API sends a **sequence of events** (tokens or structured deltas) as generation progresses, instead of one final blob.

## Core ideas

| Idea | Meaning |
|------|--------|
| **Latency vs perceived speed** | Total time may be unchanged, but **time-to-first-token** and progressive display make the app feel responsive. |
| **Events** | Each SSE-style chunk describes lifecycle: message start, block start, **text deltas**, block stop, message stop, etc. |
| **Two layers** | (1) **Raw stream** — iterate `event` objects and filter types. (2) **SDK helper** — iterate only **text** (Python: `text_stream`). |
| **History** | Streaming is for **display**; you still need the **assembled message** to append the assistant turn to `messages` for the next request. |

## How it works

### 1. Basic streaming: `stream=True` on `create`

Pass **`stream=True`** to `client.messages.create`. The return value is an **iterable of events** (not a single `Message` object).

```python
stream = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=messages,
    stream=True,
)

for event in stream:
    print(event)
```

Use this when you need **full visibility** into every event type for logging, custom UI, or non-text content.

### 2. Understanding stream events (conceptual names)

When streaming is enabled, the API emits multiple **event types** along the lifecycle of one assistant message. Names in docs/SDKs follow this pattern (exact class names may vary slightly by SDK version):

| Event (conceptual) | Role |
|--------------------|------|
| **Message start** | A new assistant message is beginning. |
| **Content block start** | A new block is starting (text, tool use, etc.). |
| **Content block delta** | **Chunks of generated content** — for text, this is where **token/text pieces** appear. |
| **Content block stop** | The current block finished. |
| **Message delta / message stop** | Message-level completion signals. |

For **plain text display**, the pieces you usually care about live in **content block delta** events (text deltas). Showing **every** raw event is noisy for end users.

### 3. Simplified text streaming (Python SDK)

The Python SDK provides a **context manager** that focuses on **text only**: `client.messages.stream(...)` and iteration over **`stream.text_stream`** (snake_case in Python; TypeScript/JavaScript SDKs often use `textStream`).

```python
with client.messages.stream(
    model=model,
    max_tokens=1000,
    messages=messages,
) as stream:
    for text in stream.text_stream:
        print(text, end="")
```

This avoids hand-parsing delta events when you only need to **print or forward** assistant text chunks.

### 4. Full message after streaming: `get_final_message()`

After the stream completes, retrieve the **complete** `Message`-like object so you can **store** it, read **usage**, inspect **stop reason**, or **append** the assistant turn to your conversation list.

```python
with client.messages.stream(
    model=model,
    max_tokens=1000,
    messages=messages,
) as stream:
    for text in stream.text_stream:
        # e.g. push to websocket / UI
        pass

    final_message = stream.get_final_message()
# Use final_message like a normal non-streaming response for persistence / next turn
```

Pattern: **stream for UX**, **`get_final_message()` for state**.

## Trade-offs

| Approach | Pros | Cons |
|----------|------|------|
| **Non-streaming `create`** | Simple; one object; easy to test. | Long **time-to-first-byte**; feels slow in chat UIs. |
| **Raw `stream=True` + events** | Maximum control; good for tools, multimodal, custom handling. | More code; must filter event types. |
| **`messages.stream` + `text_stream`** | Minimal code for **text** UX. | Hides details—use raw stream if you need every event. |

## Pitfalls

- **Do not lose the final message** — If you only print chunks and never call **`get_final_message()`** (or equivalent), you may **fail to persist** the assistant reply or **break multi-turn** continuity.
- **Model/SDK naming** — Python uses **`text_stream`**; other languages may use **`textStream`**. Same concept.
- **Token usage** — Streaming does not remove **output token** costs; it changes **delivery**, not billing semantics (confirm current billing details in official docs).

## Interview angles

- **Why stream?** Perceived latency, chat UX, progressive rendering; optionally mention **SSE** or event streams.
- **What is in a delta?** Incremental **content** (often text chunks); full message is **assembled** at the end.
- **How do you keep conversation state?** After streaming, use the **final message object** (or its structured content) as the assistant turn in `messages`.

## Quick recap

- Add **`stream=True`** to **`messages.create`** to iterate **raw events**, or use **`client.messages.stream`** and **`text_stream`** for **text-only** streaming in Python.
- **Text the user sees** incrementally comes from **delta** / **`text_stream`** chunks; **lifecycle events** are for structure and tooling.
- Call **`get_final_message()`** after streaming to get the **complete assistant message** for **storage** and **next-turn** context.

## See also

- [multi_turn_conversations.md](./multi_turn_conversations.md) — building `messages` across turns.
- [python_sdk_first_request.md](./python_sdk_first_request.md) — baseline `messages.create` setup.

_Exact event type names and method names can vary slightly by SDK version; verify against [Anthropic API docs](https://docs.anthropic.com/) for your SDK._
