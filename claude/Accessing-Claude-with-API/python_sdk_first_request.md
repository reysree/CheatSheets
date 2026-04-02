# Claude API — first request in Python (Jupyter, `.env`, SDK)

Practical steps to call the **Messages API** from a **Python** environment (e.g. Jupyter): secrets, packages, client setup, `client.messages.create`, and reading the assistant reply.

## Overview

You need: a **secret API key** (never committed), **packages** (`anthropic`, `python-dotenv`), **environment loading**, an **Anthropic client**, a **model id**, and a **`messages.create`** call with **`model`**, **`max_tokens`**, and **`messages`**. The response is a rich object; for plain text you usually read the first **text** block from **`content`**.

## Environment setup

| Step | What to do |
|------|------------|
| **Jupyter** | Use a notebook (or any Python env) where you can run cells and install packages. |
| **`.env` file** | At project root (or a path you load explicitly), store `ANTHROPIC_API_KEY="your-key-here"` — value in **double quotes** is a common style; the important part is **`load_dotenv()`** sees the variable name your code expects (often `ANTHROPIC_API_KEY`). |
| **Install packages** | `pip install anthropic python-dotenv` (and Jupyter stack if needed: e.g. `jupyter` or use VS Code / Cursor notebook support). |
| **Git** | Add **`.env`** to **`.gitignore`** so the API key is **not** pushed to GitHub or any remote. Treat any file that holds secrets the same way. |

## Python: load env and create the client

```python
from dotenv import load_dotenv
from anthropic import Anthropic

load_dotenv()  # loads variables from .env into the process environment

client = Anthropic()  # reads ANTHROPIC_API_KEY from the environment by default
```

Pick a **model id** string that matches what your Anthropic project supports (Console / [API docs](https://docs.anthropic.com/) — ids are versioned, e.g. Sonnet family names change over time):

```python
model = "claude-sonnet-4-20250514"  # example; replace with current id from docs / console
```

_Use the exact string from Anthropic’s model list; “Claude Sonnet 4” in docs maps to a specific id like the above pattern._

## Sending a message: `client.messages.create`

The client method is **`client.messages.create`**. Conceptually it needs at least:

| Parameter | Role |
|-----------|------|
| **`model`** | Which Claude model runs this request (string id). |
| **`max_tokens`** | **Upper bound on output length** (in tokens) for this turn. Too small → truncated or oddly clipped answers; too large → unnecessary headroom. Tune for the task. |
| **`messages`** | A **list** of turns. Each item is a **dict** (or structured object) with **`role`** and **`content`**. |

### Roles: `user` vs `assistant`

- **`user`** — what **you** send in (the prompt, question, instructions).
- **`assistant`** — what the **model** returned in **earlier** turns when you are **continuing a conversation** (you pass prior assistant text back so the model sees full context).

For a **single-shot** question you often send **one** `user` message only.

### Example call

```python
message = client.messages.create(
    model=model,
    max_tokens=1000,
    messages=[
        {
            "role": "user",
            "content": "What is quantum computing?",
        },
    ],
)
```

## Reading the reply

The returned **`message`** object includes **`content`**: a **list of blocks** (text, tool use, etc.). For a normal text reply, the first block is often a **text** block.

**Anthropic Python SDK:** text is on the block’s **`.text`** attribute (not `text_content`):

```python
# Typical pattern for a single text block:
text = message.content[0].text
```

If you might get multiple blocks or non-text blocks, loop and filter by `type == "text"` (see [Messages API](https://docs.anthropic.com/en/api/messages)).

## Pitfalls

- **Key in repo** — Committing `.env` or pasting keys into notebooks that get shared → rotate the key and use `.gitignore` + env/secrets.
- **`max_tokens` too low** — Looks like a “bad model” but is often **cutoff**; raise limit or shorten the task.
- **Wrong model string** — Copy model ids from current docs; old tutorials go stale.
- **Attribute name** — Use **`.text`** on text blocks in the official Python SDK; if a course says `text_content`, verify against the SDK version you installed.

## Interview angles

- *“How do you avoid leaking the Anthropic key?”* → Env file + `.gitignore`, or secret manager; never embed in client-side code for production.
- *“What does `max_tokens` bound?”* → **Output** tokens for that response; related but distinct from **context window** / input size.
- *“What do you pass to `messages.create`?”* → At minimum **model**, **max_tokens**, **messages** (roles + content).

## Quick recap

- **Setup:** `.env` with `ANTHROPIC_API_KEY`, `pip install anthropic python-dotenv`, **`load_dotenv()`**, **`Anthropic()`** client.
- **Call:** `client.messages.create(model=..., max_tokens=..., messages=[{"role": "user", "content": "..."}])`.
- **Read text:** `message.content[0].text` for the first text block (confirm block type in robust code).
- **Safety:** `.env` in **`.gitignore`**; confirm **model id** in current docs.

## References

- [Anthropic API documentation](https://docs.anthropic.com/) — authentication, Messages API, model ids.
- [anthropic-sdk-python](https://github.com/anthropics/anthropic-sdk-python) — install and response types (verify `content` block shapes for your version).
