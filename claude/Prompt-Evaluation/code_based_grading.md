# Code-based grading (syntax, format, hybrid with model judge)

Study notes on **programmatic** checks for **code-generating** models: enforce **output shape**, verify **parseable** Python / JSON / regex, then **combine** with a **model grader** for **task following**. Complements `prompt_evaluation_graders.md` (overview of grader types).

**Related:** `eval_dataset_generation_example.md` (dataset shape—add a **`format`** field); `core_evaluation_pipeline.md` (`run_test_case`); `iterative_eval_workflow.md` (track **mean** score over iterations).

---

## Overview

For codegen evals, “sounds right” is not enough—you want **objective** signals that the artifact is the **right kind** of blob and **actually parses**.

| Criterion | Typical grader |
|-----------|----------------|
| **Format** | Response is **only** the requested type (Python / JSON / regex), minimal prose | **Code** (heuristics, stripping fences, keywords) |
| **Valid syntax** | Output **parses** as JSON, **AST-parseable** Python, or **compilable** regex | **Code** |
| **Task following** | Answer matches the **task** and is **correct** | **Model** (or human)—semantic |

**Code** handles **format + syntax** well; **task following** is left to a **model grader**. **Together** they give a more **complete** picture than either alone.

---

## Syntax validation helpers

Try **parse** the model output; map success/fail to a **numeric** score. Here **binary** scores: **10** if valid, **0** if not (you can switch to partial credit or 0–1 scaled averages if you prefer).

```python
import ast
import json
import re

def validate_json(text):
    try:
        json.loads(text.strip())
        return 10
    except json.JSONDecodeError:
        return 0

def validate_python(text):
    try:
        ast.parse(text.strip())
        return 10
    except SyntaxError:
        return 0

def validate_regex(text):
    try:
        re.compile(text.strip())
        return 10
    except re.error:
        return 0
```

**Reality check:** If the model wraps output in markdown **fences**, `.strip()` is not enough—often you must **strip** leading/trailing ```…``` blocks before parsing.

---

## Dataset: expected `format` per case

The code grader needs to know **which** validator to run. Add a field on each test case:

```json
{
  "task": "Create a Python function to validate an AWS IAM username",
  "format": "python"
}
```

Allowed values might be `"python"`, `"json"`, `"regex"`—keep them **stable** and document them for **dataset generation** (extend the meta-prompt in `eval_dataset_generation_example.md` so generated rows include **`format`**).

### Dispatcher example

```python
def grade_syntax(output, test_case):
    fmt = (test_case.get("format") or "").lower()
    text = output  # optionally strip markdown fences here
    if fmt == "json":
        return validate_json(text)
    if fmt == "python":
        return validate_python(text)
    if fmt == "regex":
        return validate_regex(text)
    return 0  # unknown format: fail closed or raise
```

---

## Prompt clarity (reduces format failures)

Tight **user-facing** instructions improve both **human** readability and **grader** hit rate:

- Respond **only** with Python, JSON, or a plain regex—no commentary.
- **No** leading explanation or trailing notes.

**Optional steer:** prefill the assistant with a **generic** code fence so the model starts in a “code block” mindset:

```python
fence = "`" * 3
add_assistant_message(messages, fence + "code")
```

(Use `fence + "python"` or `fence + "json"` instead if you want a specific fence label; match what your **strip** logic expects.)

---

## Combining code score with model score

**Model** judge captures **task following** and quality; **code** judge captures **syntax** (and you can add **format** heuristics). A simple **fusion** is an unweighted **average**:

```python
model_grade = grade_by_model(test_case, output)
model_score = model_grade["score"]
syntax_score = grade_syntax(output, test_case)
score = (model_score + syntax_score) / 2
```

Adjust **weights** if **correctness** vs **style** matters more for your product, e.g. `0.7 * model_score + 0.3 * syntax_score`.

Return **`reasoning`** (and optional **syntax breakdown**) in `run_test_case` for debugging.

---

## Testing and iteration

Run the full eval to get a **baseline** **mean** score. The absolute number is less important than **direction**: same **dataset**, same **grader** code, **prompt** changes—does the **average** move the right way?

That turns prompt work into a **quantitative** loop instead of purely subjective “looks better.”

---

## Trade-offs and pitfalls

- **Binary 10/0 syntax** — Harsh; one stray character fails; consider normalization (strip fences) before parse.
- **Regex `compile`** — Validates **pattern** syntax, not that it matches the right strings.
- **`ast.parse`** — Confirms Python **syntax**, not **runtime** correctness or imports.
- **Double API cost** — Generation + model grader + (cheap) local syntax checks.
- **Format without semantics** — High **syntax** score + wrong answer is still wrong—keep the **model** (or tests) in the loop.

---

## Interview / task angles

- **What can code graders verify cheaply?** — Parseability, structure, simple rules.
- **What needs a model grader?** — Whether the code **does** what the task asked.
- **Why combine scores?** — **Technical validity** and **semantic quality** are different axes.
- **Why put `format` in the dataset?** — Lets one suite mix Python / JSON / regex cases with the right validator.

---

## Quick recap

1. **Code grading** — **Format** (mostly rules) + **valid syntax** (parse/compile).  
2. **Dataset** — Include **`format`** so `grade_syntax` dispatches correctly.  
3. **Prompt** — Demand **raw** artifact; optional **prefill** with a code fence.  
4. **Hybrid score** — e.g. **average** of **model** and **syntax** scores; tune weights.  
5. **Iterate** — Use **mean** on a **fixed** suite to measure prompt changes.
