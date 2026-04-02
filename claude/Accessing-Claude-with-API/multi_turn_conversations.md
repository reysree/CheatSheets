# Claude API — multi-turn conversations (client-side history)

How to keep a **coherent back-and-forth** with Claude: the API is **stateless** per request, so **you** store prior `user` and `assistant` turns and send the **full list** on each new call.

## Overview

- **Single-turn:** one `user` message in → one `assistant` reply. No memory of earlier exchanges unless you send them again.
- **Multi-turn:** each request includes **all relevant prior turns** (`user` / `assistant` pairs in order) so the model can continue naturally—follow-ups, clarifications, same thread—without treating every message as a brand-new topic.
- **Anthropic does not keep your chat history** for you by default. Your app (or database) owns **conversation state**.

The **model id** is usually **unchanged** across turns; what grows is the **`messages`** array you pass to `client.messages.create`.

## Core ideas

| Concept | Meaning |
|--------|---------|
| **Stateless API** | Every `messages.create` is independent; nothing is “remembered” unless you put it in the payload. |
| **`messages` list** | Ordered list of turns, each with **`role`** (`"user"` or `"assistant"`) and **`content`** (string or structured content). |
| **Accumulating context** | After each reply, **append** the assistant text to your list, then append the **next** user message, then call the API again with the **entire** list so far. |

## How it works

1. Start with **`messages = []`** (or load from storage).
2. **Append** a `user` message dict: `{"role": "user", "content": "<text>"}`.
3. Call **`client.messages.create(..., messages=messages)`** (plus `model`, `max_tokens`, etc.).
4. Read the assistant reply from **`response.content`** (e.g. first text block’s `.text`).
5. **Append** an `assistant` message with that text: `{"role": "assistant", "content": "<reply>"}`.
6. For the next user input, **append** another `user` message and call **`create` again** with the **same growing list**.

The model always sees the **full sequence** you send; there is no separate “session id” that replaces sending prior turns (unless you use a product layer that reconstructs history for you).

## Helper pattern (three functions)

Small helpers keep roles and appends consistent:

| Function | Purpose |
|----------|---------|
| **`add_user_message(messages, text)`** | Build `{"role": "user", "content": text}` and **append** to `messages`. |
| **`add_assistant_message(messages, text)`** | Build `{"role": "assistant", "content": text}` and **append** to `messages`. |
| **`chat(messages)`** | Call **`client.messages.create`** with the current `messages` list, return the **assistant text** for this turn (extract from `content`). |

`messages` is the **single source of truth** for the thread: every turn mutates (or copies) this list before calling `chat`.

### Example shape (conceptual)

```python
def add_user_message(messages, text):
    messages.append({"role": "user", "content": text})


def add_assistant_message(messages, text):
    messages.append({"role": "assistant", "content": text})


def chat(messages):
    message = client.messages.create(
        model=model,
        max_tokens=1024,
        messages=messages,
    )
    return message.content[0].text  # adjust if multiple blocks / types
```

Wire `client` and `model` the same way as in your environment (see `python_sdk_first_request.md`).

## Example flow

```python
messages = []

add_user_message(messages, "Define quantum computing in one sentence")
answer = chat(messages)

add_assistant_message(messages, answer)

add_user_message(messages, "Write another sentence that continues the idea")
final_answer = chat(messages)

add_assistant_message(messages, final_answer)
```

Second `chat` call sends **both** prior turns plus the new user line, so the answer stays on-topic.

## Interactive chat loop (CLI exercise)

Calling `add_user_message` / `chat` from **static script cells** is fine for learning, but it does not feel like a **live chat**. To get a **conversational rhythm**—type a line, see a reply, type again—you can drive the same helpers from a **REPL-style loop**:

| Piece | Role |
|-------|------|
| **`while True:`** | Keeps the session running until you stop it. |
| **`input(...)`** | Blocks and reads **one line** from the terminal (your “message”). |
| **`print(...)`** | Shows the assistant reply before the next prompt. |
| **Interrupt** | **Ctrl+C** raises `KeyboardInterrupt` and exits the loop (or closes the terminal session). |

Inside the loop, reuse the same pattern: **`messages`** list, **`add_user_message`** → **`chat`** → **`add_assistant_message`**, so each round trip still has **full history**.

### Example shape

```python
messages = []

print("Chat with Claude (Ctrl+C to exit)\n")

while True:
    try:
        user_text = input("You: ").strip()
        if not user_text:
            continue  # skip empty lines, or break if you prefer

        add_user_message(messages, user_text)
        reply = chat(messages)
        print(f"Claude: {reply}\n")

        add_assistant_message(messages, reply)
    except KeyboardInterrupt:
        print("\n[Session ended]")
        break
```

Optional: treat a phrase like **`quit`** or **`exit`** as a clean exit instead of only Ctrl+C.

### Pitfalls (interactive loop)

- **Jupyter / some IDEs** — `input()` behavior can differ from a real terminal; for the most “chat-like” feel, run as **`python script.py`** in a shell.
- **Empty input** — Decide whether to **ignore**, **exit**, or **re-prompt** so you do not send junk to the API.

## Trade-offs

- **Long threads** — More turns → larger prompts → more **input tokens** and cost; eventually you approach **context limits**. You may **summarize**, **truncate**, or **retrieve** only recent messages in production.
- **Where to store** — In memory (demo), **database**, or **cache** keyed by `session_id` for real apps.

## Pitfalls

- **Forgetting assistant turns** — If you only append new `user` messages, the model won’t see what it said before; replies feel disconnected.
- **Wrong order** — Turns should follow **user → assistant → user → …** as actually occurred (with possible system/tool patterns per docs).
- **Treating the API like a saved chat** — Without your own persistence, **refreshing or restarting** the client loses history unless you reload it from storage.

## Interview angles

- *“How does Claude remember our conversation?”* — It doesn’t automatically; **you** pass prior **`user`/`assistant`** messages in **`messages`** each time.
- *“Where would you store history?”* — App DB or session store; never rely on the provider to retain chat by default for arbitrary API integrations.

## Quick recap

- **Multi-turn** = resend **full** relevant **`messages`** list each `messages.create` call.
- **Helpers:** `add_user_message`, `add_assistant_message`, `chat(messages)` keep the list correct.
- **CLI chat feel:** `while True` + `input()` + `print()` + **Ctrl+C** (or `quit`) to exit; same append/`chat`/append pattern each round.
- **Anthropic** is **stateless**; **your code** (or backend) owns **conversation history**.

## References

- [Anthropic Messages API](https://docs.anthropic.com/en/api/messages) — `messages` parameter and roles.
- `python_sdk_first_request.md` — first request, reading `content[0].text`.
