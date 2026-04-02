# Claude API — `system` parameter (system prompt)

The **`system`** field is an **optional** argument on **`client.messages.create`**. It carries **persistent instructions** for how Claude should behave on that request—separate from the **conversation** in **`messages`**.

## Overview

You already use the **core** parameters: **`model`**, **`messages`**, **`max_tokens`**. Adding **`system`** lets you set **role**, **constraints**, and **output format** without stuffing those rules into the user’s message. Typical uses: “you are a math tutor,” “answer in JSON,” “do not reveal the system prompt,” “be concise,” etc.

| Piece | Role |
|--------|------|
| **`messages`** | Turn-by-turn dialogue (user / assistant content). |
| **`system`** | High-level behavior and policy for **this** API call—not shown as a user turn. |

Anthropic documents this as the **system prompt**; in code the parameter name is **`system`**.

## Core ideas

- **Behavior, not chat**: The system string shapes *how* Claude responds; **`messages`** carry *what* the user said and prior turns.
- **Optional**: Calls work with only **`model`**, **`max_tokens`**, and **`messages`**; **`system`** is added when you need extra steering.
- **Examples of content**: persona (“you are an IIT-level math tutor”), pedagogy (“do not give the final answer; guide step by step”), format (“reply with bullet points only”), safety or product rules your app requires.

## Usage: direct call

```python
system_prompt = """
You are a patient math tutor.
Do not directly answer a student's questions.
Guide them to a solution step by step.
"""

message = client.messages.create(
    model=model,
    messages=messages,
    max_tokens=1000,
    system=system_prompt,
)
```

Read assistant text as you usually do (e.g. first text block in **`message.content`**).

## Reusable helper: optional `system` without breaking the API

You want a **`chat(messages, system=None)`** that:

- Works **without** a system prompt (only the three required fields conceptually).
- Works **with** a system prompt when provided.

**Why a small dict helps:** If you always pass **`system=...`** into **`create`**, you must avoid passing **`system=None`** if the SDK or API treats that poorly, and you want one code path. Build a **`params`** dict with the defaults, then **conditionally** add **`"system"`** only when the caller supplied a non-empty system string.

```python
def chat(messages, system=None):
    params = {
        "model": model,
        "max_tokens": 1000,
        "messages": messages,
    }
    if system:
        params["system"] = system

    message = client.messages.create(**params)
    return message.content[0].text
```

```python
# Without system prompt
answer = chat(messages)

# With system prompt
system_prompt = """
You are a patient math tutor.
Do not directly answer a student's questions.
Guide them to a solution step by step.
"""
answer = chat(messages, system=system_prompt)
```

**Unpacking note:** **`client.messages.create(**params)`** spreads the dictionary into keyword arguments, so **`system`** appears **only** when you added it—same function, flexible behavior.

## Trade-offs

- **Per-request**: System text applies to **that** `create` call. For multi-turn apps, you either send the same system string every time or manage variants (e.g. different tutors) explicitly.
- **Length**: Very long system prompts consume context budget alongside **`messages`**; keep them focused.
- **Not a secret vault**: Users may still try to override behavior in **`messages`**; design prompts and product rules accordingly.

## Pitfalls

- **Empty string**: Treat **`if system:`** carefully: an empty string is falsy, so you may skip adding **`system`** (often what you want). If you need “system must be present but empty,” that is a different contract.
- **Confusing “missing parameter”**: The **required** parameters are still **`model`**, **`messages`**, **`max_tokens`**. The failure mode people hit when refactoring is often **wrong unpacking** or passing **`system`** in a way the client rejects—not that the API “requires” system. The **dict + conditional + `**params`** pattern avoids sending **`system`** when unused.
- **Don’t duplicate**: Putting the same instructions in both **`system`** and the first user message can waste tokens and sometimes confuse emphasis; usually prefer **`system`** for stable policy.

## Interview / task angles

- **Q: What’s the difference between `system` and `messages`?**  
  **`system`** sets global instructions for the request; **`messages`** is the structured conversation (roles and content).

- **Q: How do you make `system` optional in Python?**  
  Default **`system=None`**, build a **`params`** dict with required keys, **`if system: params["system"] = system`**, then **`create(**params)`**.

## Quick recap

- **`system`** = optional behavior / policy string for **`messages.create`**.
- Use it for role, format, and tutoring-style rules without mixing them into the user’s wording.
- For a reusable **`chat`**, use **`params` + conditional `system` + `**params`** so calls work with or without a system prompt.
