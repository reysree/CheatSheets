# Claude API — `temperature` (sampling creativity vs. determinism)

Study notes on the optional **`temperature`** parameter: how it relates to **next-token probabilities**, what low vs. high values do, how to pick a range for a task, and how to set it in Python.

## Overview

**Temperature** controls how **sharply** the model samples from its **distribution over next tokens**. It does **not** change the underlying model weights for a single request; it changes **how** the API turns logits into a choice for each new token.

- **Low temperature** → sampling stays **close to the most likely** tokens → outputs feel **more predictable**, **consistent**, and **factual** when the task has a clear “best” continuation.
- **High temperature** → the probability mass **spreads** across more tokens → **more variety** and **creativity**, with higher chance of **less likely** (but still plausible) continuations.

Use it deliberately: the right value depends on whether the task rewards **one correct-ish answer** or **exploration and phrasing diversity**.

## How it connects to prior concepts (tokenization → next token)

From the end-to-end model picture (see `api_usage.md`):

1. Text is **tokenized**; the model builds **contextual** representations.
2. At each step, the model assigns **scores** to candidate next tokens; these become a **probability distribution** over the vocabulary (after softmax).
3. **Sampling** picks the next token **from** that distribution. Without extra controls, you might always take the single highest-probability token (**greedy** / **argmax** behavior).

**Temperature** scales how “flat” or “peaked” that distribution is before sampling:

| Intuition | Effect on probabilities | Typical feel of output |
|-----------|-------------------------|-------------------------|
| **Lower** | Mass concentrates on **top** candidates | Stable, repeatable, “safest” wording |
| **Higher** | Mass **spreads** to lower-ranked candidates | More varied, surprising, sometimes less on-rails |

So: the model still only proposes **plausible** tokens; temperature adjusts **how often** you pick slightly less likely ones, not magic new facts.

## Choosing a temperature (rule of thumb by task)

These **bands** are **heuristic**—always validate on **your** prompts, model id, and evals. Other parameters (**`top_p`**, system prompt, etc.) also matter.

| Band | Example range | When it tends to help |
|------|----------------|------------------------|
| **Low** | **0.0 – 0.3** | **Factual** answers, **coding**, **data extraction**, **classification**, **content moderation**, anything where you want **minimal** creative drift |
| **Medium** | **0.4 – 0.7** | **Summaries**, **teaching** explanations, **problem-solving** where a bit of paraphrase or structure variety is useful |
| **High** | **~0.8 – 1.0** | **Brainstorming**, **creative writing**, **marketing** copy, **jokes**, ideation where **diversity** of phrasing or ideas is the goal |

**Creative vs. non-creative framing:** ask “Do I want the **same** kind of answer every time (low), or **many different** reasonable answers across runs (high)?”

## API / Python usage

Pass **`temperature`** on `client.messages.create` (alongside `model`, `max_tokens`, `messages`, etc.):

```python
message = client.messages.create(
    model=model,
    max_tokens=1000,
    temperature=0.2,  # e.g. factual / code-ish tasks
    messages=[
        {"role": "user", "content": "..."},
    ],
)
```

**Typical numeric range:** **0.0 to 1.0** in Anthropic’s Messages API (confirm **exact** allowed range and defaults in the [current API docs](https://docs.anthropic.com/)).

## Pitfalls and misconceptions

- **High temperature does not guarantee “random nonsense.”** It **increases the chance** of sampling **less likely** tokens; outputs remain **language-model** outputs, bounded by training and context.
- **Low temperature (e.g. 0.0) does not guarantee identical outputs** across runs if other sources of variability exist (nondeterminism in infrastructure, concurrent changes, or other parameters)—but it **reduces** sampling variety step by step.
- **Temperature is not a substitute for instructions.** A vague prompt stays vague; tighten the **system prompt** and **user message** first, then tune temperature.
- **Interaction with `top_p` (and similar):** both reshape sampling; changing one can change the effective behavior of the other—tune them together for critical pipelines.

## Interview / task angles

- *“What does temperature do?”* → It shapes **next-token sampling** from the model’s distribution: lower → more **deterministic** / peaky; higher → more **diverse** continuations.
- *“When would you use low vs. high?”* → Low for **reliability and precision** (code, extraction); high for **brainstorming and creative** generation.
- *“Does high temperature make the model smarter?”* → No—it changes **variation**, not **ground truth** or **reasoning quality** by itself.

## Quick recap

- **Temperature** = control over **how peaked** the next-token distribution is when **sampling**.
- **Low** → favor **highest-probability** continuations; **high** → **more spread**, more **creative** variation.
- Pick by task: **factual / code / extraction** → lower; **ideation / creative writing** → higher.
- Set via **`temperature`** on the API call; **validate** on real prompts; remember **`top_p`** and prompts **co-determine** behavior.

## See also

- `api_usage.md` — tokenization, contextualization, and generation loop (`temperature` mentioned in the generation phase).
- `python_sdk_first_request.md` — `client.messages.create` basics.

## References

- [Anthropic API documentation](https://docs.anthropic.com/) — parameter defaults, ranges, and model-specific notes.
