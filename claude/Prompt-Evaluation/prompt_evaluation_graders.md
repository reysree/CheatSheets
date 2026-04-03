# Prompt evaluation graders (code, model, human)

Study notes on **grading** model outputs: turn each response into a **measurable signal** (often a **1–10** score where **10** is high quality). Aligns with `Claude/course_video_reference.md` (*Model-Based Grading*, *Code-Based Grading*).

**Related:** `core_evaluation_pipeline.md` (where `run_test_case` calls the grader); `iterative_eval_workflow.md` (average score across cases, iteration). **Code grading (JSON/Python/regex syntax, `format` field, hybrid scores):** `code_based_grading.md`.

---

## Overview

A **grader** takes **model output** (and usually **context**: task, rubric, constraints) and returns feedback—typically a **numeric score** plus optional **structured** fields. Graders give **objective-ish** signals you can **aggregate** (mean, percentiles) and **track** across prompt versions.

**Caveat:** No grader is perfect—**code** graders miss semantics; **model** graders can drift or cluster around middling scores; **human** graders do not scale. Combine approaches where it matters.

---

## Types of graders

| Type | Idea | Typical uses |
|------|------|----------------|
| **Code** | Programmatic checks you define | Length, allow/deny words, JSON/Python/regex **syntax**, simple heuristics, readability metrics |
| **Model** | Second **API call** with an **eval prompt** | Response quality, **instruction following**, completeness, helpfulness, safety, **task fit** when rules are fuzzy |
| **Human** | Manual review | Nuance, “does this feel right?”, edge cases; **expensive** and slow |

**Code** — Must return a **usable signal** (often scaled **1–10** or binary pass/fail mapped to a scale).

**Model** — Very **flexible** for semantic criteria; treat as a **baseline** you sanity-check (variance, bias toward ~6–7).

**Human** — Most **flexible**; use for **calibration**, **spot checks**, or gold labels—not for every case at scale.

---

## Defining evaluation criteria first

Before implementing, write **explicit criteria**. Example for **code-generation** style tasks:

| Criterion | What it checks | Often graded with |
|-----------|----------------|-------------------|
| **Format** | Output is **only** Python / JSON / regex as required—no extra explanation | **Code** (structure, markers) + sometimes **model** |
| **Valid syntax** | Parses as valid Python / JSON; regex compiles | **Code** |
| **Task following** | Solution **addresses** the user’s task with **accurate** logic | **Model** (or human)—hard to fully capture in rules |

**Format** and **syntax** map cleanly to **code** graders. **Task following** usually needs a **model** or **human** judge.

---

## Implementing a model grader

Use a **separate** evaluation prompt. Ask for **structured JSON** with **reasoning**, not **only** a score—otherwise the judge often **defaults to middling scores** (e.g. around **6**) with little discrimination.

```python
import json

def grade_by_model(test_case, output):
    task = test_case["task"]
    solution = output
    eval_prompt = f"""
You are an expert code reviewer. Evaluate this AI-generated solution.

Task: {task}
Solution: {solution}

Provide your evaluation as a structured JSON object with:
- "strengths": An array of 1-3 key strengths
- "weaknesses": An array of 1-3 key areas for improvement
- "reasoning": A concise explanation of your assessment
- "score": A number between 1-10
"""
    messages = []
    add_user_message(messages, eval_prompt)
    fence = "`" * 3
    add_assistant_message(messages, fence + "json")
    eval_text = chat(messages, stop_sequences=[fence])
    return json.loads(eval_text)
```

**Pattern:** **Prefill** the assistant with an opening **JSON** fence + **stop** on the closing fence—same idea as dataset generation (`eval_dataset_generation_example.md`). Validate / repair JSON in production.

---

## Integrating into `run_test_case` and `run_eval`

Replace the placeholder score with the grader output; keep **reasoning** (and optionally strengths/weaknesses) for debugging.

```python
def run_test_case(test_case):
    output = run_prompt(test_case)
    model_grade = grade_by_model(test_case, output)
    score = model_grade["score"]
    reasoning = model_grade["reasoning"]
    return {
        "output": output,
        "test_case": test_case,
        "score": score,
        "reasoning": reasoning,
    }
```

**Aggregate** with a **mean** (or weighted mean) over cases:

```python
from statistics import mean

def run_eval(dataset):
    results = []
    for test_case in dataset:
        results.append(run_test_case(test_case))
    average_score = mean(r["score"] for r in results)
    print(f"Average score: {average_score}")
    return results
```

Use the **average** as one **headline metric** while iterating the **user-facing** prompt—same dataset, same grader, compare **before/after** (`iterative_eval_workflow.md`).

---

## Trade-offs and pitfalls

- **Model grader inconsistency** — Same output might get different scores across runs; reduce temperature for the judge, fix rubric wording, or average multiple judge calls if needed.
- **Score compression** — Judge gives **6** everywhere if the prompt does not force **calibration** and **spread**; strengths/weaknesses/reasoning help.
- **Overfitting the grader** — Optimizing only for what the **model judge** likes may diverge from **user** value; cross-check with **code** checks and **human** samples.
- **Cost and latency** — Two API calls per case (generate + grade); budget accordingly.

---

## Interview / task angles

- **Three grader types** — Code (rules/syntax), model (flexible semantics), human (gold / spot check).
- **Why ask for reasoning in a model grader?** — Reduces arbitrary scores and helps **debug** failures.
- **What to implement first?** — **Code** checks for format/syntax; **model** judge for task quality; **human** for calibration.
- **How to track prompt improvements?** — **Mean score** on a **fixed** suite + same grader version + note **variance**.

---

## Quick recap

1. **Graders** — Map output → **score** (often **1–10**) + optional structured fields.  
2. **Code** — Fast, deterministic; good for **format** and **syntax**.  
3. **Model** — Flexible; ask for **JSON** with **reasoning** + **score**; watch **middling** defaults and **variance**.  
4. **Human** — Flexible; does not scale.  
5. **Criteria** — Define **before** coding; match criterion to **grader type**.  
6. **Pipeline** — `run_test_case` calls **grader**; `run_eval` computes **mean** for tracking iterations.
