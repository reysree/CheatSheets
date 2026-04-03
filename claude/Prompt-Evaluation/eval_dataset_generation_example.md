# Example: generating an eval dataset (AWS coding prompt)

Study notes walking through **creating test inputs** for prompt evaluation: define the product goal, a starter prompt, a **meta-prompt** that asks the model to emit a JSON dataset, **prefill + stop sequences** for clean parsing, then **save** `dataset.json` for later runs. Pattern aligns with `Claude/course_video_reference.md` (*Generating Test Datasets*).

**Related:** `core_evaluation_pipeline.md` (`run_prompt` / `run_test_case` / `run_eval`); `prompt_evaluation_graders.md` (code / model / human graders); `code_based_grading.md` (add **`format`** per row for syntax validators); `iterative_eval_workflow.md` (full eval loop); `prompt_engineering_and_evaluation.md`.

---

## Overview

Building a custom eval workflow starts with a **clear prompt** and **inputs** that stress what you care about. This example evaluates a helper that must return **only** usable artifacts—**Python**, **JSON**, or **regex**—for **AWS**-style tasks, with **no** extra explanations, headers, or footers.

---

## Setting the goal

| Output type | Constraint |
|-------------|------------|
| Python | Typically a small function or snippet |
| JSON | Configuration-style single object |
| Regex | Single pattern where appropriate |

**Requirement:** For each user “task,” the model should return **clean output** in exactly one of these forms—no chatter around it.

**Starter prompt (version 1)** — minimal; you will tighten after measuring:

```python
prompt = f"""
Please provide a solution to the following task:
{task}
"""
```

---

## What goes in the eval dataset

- **Inputs only** (for this step): an **array of JSON objects**, each with at least a **`task`** field describing what Claude should do.
- Later you add **expected answers** and a **grader**; here the focus is **generating diverse tasks** that match your three output types and AWS theme.
- You can **hand-write** cases or **generate** them with a model (often a **faster/cheaper** model like **Haiku** for bulk generation).

---

## Generating test data with code

### Message helpers and `chat`

Typical small helpers to build `messages` and call `client.messages.create`:

```python
def add_user_message(messages, text):
    messages.append({"role": "user", "content": text})

def add_assistant_message(messages, text):
    messages.append({"role": "assistant", "content": text})

def chat(messages, system=None, temperature=1.0, stop_sequences=None):
    if stop_sequences is None:
        stop_sequences = []
    params = {
        "model": model,
        "max_tokens": 1000,
        "messages": messages,
        "temperature": temperature,
    }
    if system:
        params["system"] = system
    if stop_sequences:
        params["stop_sequences"] = stop_sequences
    response = client.messages.create(**params)
    return response.content[0].text
```

Use a **Haiku-class** model for `generate_dataset` if you want speed and lower cost for synthetic tasks.

### Meta-prompt + JSON prefill + stop

Ask the model to emit **only** a JSON array of tasks, with constraints so cases stay **small** and **single-artifact**:

```python
import json

def generate_dataset():
    prompt = """
Generate an evaluation dataset for a prompt evaluation. The dataset will be used to evaluate prompts that generate Python, JSON, or Regex specifically for AWS-related tasks. Generate an array of JSON objects, each representing a task that requires Python, JSON, or a Regex to complete.

Example output shape (valid JSON only, no markdown fences):
[
  { "task": "Description of task" },
  ...
]

* Focus on tasks that can be solved by writing a single Python function, a single JSON object, or a single regex
* Focus on tasks that do not require writing much code

Please generate 3 objects.
"""
    messages = []
    add_user_message(messages, prompt)
    fence = "`" * 3
    add_assistant_message(messages, fence + "json")  # prefill: ```json
    text = chat(messages, stop_sequences=[fence])
    return json.loads(text)
```

**Why prefill + stop?** The assistant message **starts** with an opening markdown-style fence plus `json`, so the model continues with the raw JSON array; **`stop_sequences`** on the **closing** fence keeps the returned text easy to parse with `json.loads` (confirm what your SDK returns—sometimes you strip the prefill prefix before parsing).

### Run and save

```python
dataset = generate_dataset()
print(dataset)

with open("dataset.json", "w", encoding="utf-8") as f:
    json.dump(dataset, f, indent=2)
```

You now have a **reusable** list of tasks for batching your evaluation prompt and comparing versions.

---

## Trade-offs

| Choice | Upside | Downside |
|--------|--------|----------|
| **Model-generated** tasks | Fast coverage, varied wording | May miss nasty edge cases; may drift from real user phrasing |
| **Haiku** for generation | Cheap / fast | May need human spot-check or Sonnet pass for quality |
| **Small N** (e.g. 3) for demo | Quick to iterate | Not enough for confident scores—grow the suite over time |

---

## Pitfalls

- **Invalid JSON** — Model adds prose or truncates; wrap `json.loads` in error handling or repair logic; tighten the meta-prompt.
- **Mismatched schema** — If downstream code expects extra keys (`type`, `expected_format`), encode that in the meta-prompt and validate.
- **Eval set = training set** — Synthetic tasks that are too easy or too similar to the meta-prompt can **inflate** scores; mix in real or adversarial cases later.

---

## Interview / task angles

- **Why generate datasets with a model?** — Scale and diversity of inputs; human writes the **meta-prompt** and validates samples.
- **Prefill + stop for JSON** — Reduces markdown noise and bounds the parse target (same pattern as structured output in the course).
- **Next steps after `dataset.json`?** — Run each `task` through your **real** system prompt → collect outputs → **grade** vs rubric or gold → average → iterate prompt (see `iterative_eval_workflow.md`).

---

## Quick recap

1. **Goal** — Three output families (Python / JSON / regex), AWS context, **raw** output only.  
2. **Dataset** — JSON array of `{ "task": "..." }` (extend schema as needed).  
3. **Generation** — Meta-prompt + **assistant prefill** ` ```json ` + **stop** ` ``` ` → `json.loads`.  
4. **Persist** — `dataset.json` for repeatable eval runs.  
5. **Iterate** — Expand N, add gold labels and graders, avoid **only** synthetic easy cases.
