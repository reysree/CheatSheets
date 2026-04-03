# Core evaluation pipeline (`run_prompt` → `run_test_case` → `run_eval`)

Study notes on **running** an eval: merge each **test case** into a **prompt template**, call **Claude**, collect **structured results**, and leave a **hook** for grading. Matches the *Running the Eval* pattern in `Claude/course_video_reference.md` (`run_prompt`, `run_test_case`, `run_eval`).

**Prerequisite:** `dataset.json` (see `eval_dataset_generation_example.md`). **Next:** **Graders** replace the placeholder score—see `prompt_evaluation_graders.md`; aggregate scores feed **iteration** in `iterative_eval_workflow.md`.

---

## Overview

**Workflow:** Load the dataset → for each case, **instantiate** the prompt with that case → **send** messages to the model → **record** output + case + (eventually) **score**.

Three layers keep responsibilities clear:

| Function | Role |
|----------|------|
| **`run_prompt(test_case)`** | Build the user message from the template + case; call `chat`; return **model output**. |
| **`run_test_case(test_case)`** | Run `run_prompt`, then **grade** (placeholder at first); return a **result object**. |
| **`run_eval(dataset)`** | Loop all cases; return a **list of results**. |

---

## Building the core functions

Assume you already have `add_user_message`, `add_assistant_message`, and `chat` (see `eval_dataset_generation_example.md`).

### `run_prompt`

Merges the **prompt template** with the case (here, `test_case["task"]`) and returns the assistant text:

```python
def run_prompt(test_case):
    """Merge prompt and test case input; return model output."""
    prompt = f"""
Please solve the following task:

{test_case["task"]}
"""
    messages = []
    add_user_message(messages, prompt)
    output = chat(messages)
    return output
```

Early on the template is **intentionally minimal**—no strict format rules—so outputs are often **verbose**. That gap is what you **measure** and then **fix** by iterating the prompt (and grader).

### `run_test_case`

Orchestrates **one** case: run → grade → package:

```python
def run_test_case(test_case):
    """Call run_prompt, then grade the result."""
    output = run_prompt(test_case)

    # Placeholder until you implement grading
    score = 10

    return {
        "output": output,
        "test_case": test_case,
        "score": score,
    }
```

The **`score = 10`** stub lets you **wire and test** the pipeline end-to-end before investing in graders.

### `run_eval`

Runs the **full suite**:

```python
def run_eval(dataset):
    """Run run_test_case for every item in the dataset."""
    results = []
    for test_case in dataset:
        result = run_test_case(test_case)
        results.append(result)
    return results
```

---

## Running the evaluation

```python
import json

with open("dataset.json", "r", encoding="utf-8") as f:
    dataset = json.load(f)

results = run_eval(dataset)
```

**Expect latency** — Even on a fast model (e.g. Haiku), a modest dataset can take **tens of seconds** or more (sequential calls, tokens, network). Batching, concurrency, and caching are later optimizations; first get **correctness** of the loop and result shape.

---

## Examining results

Each result is one object per test case:

| Field | Meaning |
|--------|--------|
| **`output`** | Full model response for that case. |
| **`test_case`** | Original input object (e.g. `{ "task": "..." }`). |
| **`score`** | Evaluation score (stub until real grading). |

Pretty-print:

```python
print(json.dumps(results, indent=2))
```

With a **minimal** prompt, **`output`** will often include **extra explanation**—that is useful signal: your **prompt** and/or **grader** should align with the product requirement (e.g. “code only, no prose”).

---

## What this accomplishes (and what is missing)

**Done:** End-to-end path from **dataset** → **Claude** → **structured list** ready for aggregation (mean score, pass rate, exports).

**Not done yet:** **Real grading**—the hardcoded score must become **rules**, **model-as-judge**, **human spot checks**, or a **hybrid**. That is where most eval systems spend design and maintenance effort.

**Mindset:** Most “eval pipeline” codebases are **this shape**; complexity grows in **prompt quality**, **grader fidelity**, **regression tracking**, and **performance**—not in the outer loop.

---

## Trade-offs and pitfalls

- **Stub score** — Lets you test plumbing but **hides** quality; do not treat `10` as “pass” in reports.
- **Sequential `for` loop** — Simple and debuggable; slow at scale (parallelize carefully with rate limits).
- **Prompt in `run_prompt` only** — If you add **system** prompts, tools, or RAG, fold them in here (or one layer above) so every case sees the **same** stack.
- **Verbose outputs** — Often means the **instruction** is underspecified; fix with prompt iteration **and** grader criteria that match what users need.

---

## Interview / task angles

- **Why three functions?** — Separation: **generation** (`run_prompt`) vs **scoring** (`run_test_case`) vs **batching** (`run_eval`).
- **What is the minimal eval pipeline?** — Load cases → merge prompt → model → collect outputs → (later) score.
- **First-run expectations?** — Slow wall-clock on sequential API calls; placeholder scores until graders exist.

---

## Quick recap

1. **`run_prompt`** — Template + case → `chat` → string output.  
2. **`run_test_case`** — Output + grade (placeholder) → `{ output, test_case, score }`.  
3. **`run_eval`** — Map dataset → list of results.  
4. **Run** — `json.load` → `run_eval` → inspect or aggregate.  
5. **Next** — Replace **`score`** with real **graders**; tighten **prompt** using measured failures.
