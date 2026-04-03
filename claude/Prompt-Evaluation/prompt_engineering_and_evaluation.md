# Prompt engineering and prompt evaluation (Claude)

Study notes on **how to write** prompts versus **how to measure** them, plus **how evaluation fits an engineering workflow** (smoke tests → corner cases → pipelines).

---

## Part 1 — Prompt engineering vs prompt evaluation

### Overview

- **Prompt engineering** is the **craft**: how you write and structure instructions so the model understands the task, constraints, and desired output shape.
- **Prompt evaluation** is the **measurement**: how you test prompts (often automatically) to see if they **work**—reliably and across cases—not just whether they read well.

They are **complementary**: you engineer prompts, then evaluate them; evaluation feedback drives the next round of engineering.

### Side-by-side

| | Prompt engineering | Prompt evaluation |
|---|---------------------|-------------------|
| **Goal** | Shape behavior: clarity, format, examples, structure | Verify behavior: quality, regressions, comparisons |
| **Question** | “How should I ask?” | “Does this prompt actually perform?” |
| **Typical artifacts** | System/user prompts, few-shot blocks, schemas, tags | Test sets, expected outputs, metrics, A/B runs |
| **Mindset** | Design and iteration by reading/editing | Experimentation and scoring on fixed inputs |

### Prompt engineering (toolkit)

Prompt engineering is your toolkit for **crafting effective prompts** so Claude knows **what** you want and **how** to respond.

- **Multishot (few-shot) prompting** — Examples of input → desired output so the model can infer pattern, tone, and format.
- **Structuring with XML tags** (or similar delimiters) — Separate instructions, context, and user content so boundaries are unambiguous and easier to parse (by the model or downstream code).
- **Other best practices** — Clear role and task statements, explicit constraints (length, tone, “don’t do X”), output format (JSON, bullets), handling uncertainty, tool-use rules when using function calling, etc.

These improve **clarity and steerability**; they do not by themselves prove the prompt **generalizes** to all important cases.

### Prompt evaluation (measurement)

Evaluation **tests** whether the prompt (plus model, tools, retrieval) meets your bar.

- **Test against expected answers** — Gold or reference outputs, or checks (exact match, structured fields, rubric-based grading).
- **Compare versions** — Run **different versions of the same prompt** on the same inputs and compare scores or failure modes (A/B style).

Broader automation: pass/fail suites, regression runs on prompt changes, scoring pipelines—see **Part 2** below.

### Engineering vs evaluation — trade-offs and pitfalls

- **Engineering without evaluation** risks “looks good in the editor” but fails on edge cases or after model updates.
- **Evaluation without solid engineering** can yield noisy metrics if instructions are vague or examples are inconsistent.
- Do not confuse a **well-structured** prompt with a **well-performing** one—structure helps, measurement decides.
- Tiny eval sets can reward overfitting; avoid changing the prompt and the eval set in the same step without a clear hypothesis.

---

## Part 2 — Prompt evaluation in an engineering workflow

### Overview

Prompt evaluation is how you know an AI feature (system prompt + model + tools) behaves well **before** and **after** you ship changes. You typically move from **quick sanity checks** → **iterative corner-case hardening** → **automated evaluation** as confidence needs and cost allow.

### Three levels

| Level | What you do | Confidence | Cost / effort |
|--------|-------------|------------|----------------|
| **Happy-path smoke test** | Run once (or a few times) on the “obvious” scenario | Low–medium | Very low |
| **Manual / repeated testing** | Run many times; note failures; refactor the prompt for edge cases | Medium | Medium (your time) |
| **Evaluation pipeline** | Repeatable pipeline that **scores** outputs and surfaces what to fix | High | Higher (build + runs + maintenance) |

“Working” means **consistent quality** on the scenarios you care about, including **failure modes** you have explicitly tested.

### 1. Happy path (smoke test)

You build the path (system prompt, tools, retrieval). You try the **expected** user request and confirm the answer looks right.

- **Good for**: wiring, demos, first integration.
- **Limitation**: one success does not imply robustness; inputs vary.

### 2. Repeated testing and corner cases

Many runs with varied inputs surface **corner cases**: ambiguous phrasing, long context, tool errors, wrong assumptions, format drift, etc.

- **Process**: reproduce → adjust prompt (or tools/schema) → re-test (**iterate-in-the-editor** loop).

### 3. Evaluation pipeline (strongest ongoing confidence)

Automate so every change is measured the same way:

- **Dataset** — Representative prompts; **reference answers** or **rubric criteria** where possible.
- **Runs** — Batch execution against your real stack (or a close stub).
- **Scoring** — Automatic checks (exact match, JSON validity, tool calls, regex), model-as-judge (use carefully), or human samples.
- **Feedback** — Aggregate metrics (pass rate, regressions) and **per-case** notes so you know **what** to change.

**More work and cost**, but **higher confidence** after refactors, model swaps, or new features.

### Workflow trade-offs and pitfalls

- **Speed vs rigor** — Smoke tests ship fast; pipelines catch regressions.
- **Cost** — Token and engineering time; use **subset runs** in CI and full suites on release.
- **Metrics** — Align scores with product requirements; fluency alone can hide incorrectness or unsafe/tool errors.
- Avoid treating one happy-path success as “done,” overfitting to a few examples, or changing model/tools without re-running the same eval suite.

---

## Part 3 — Iterative eval dataset pipeline (typical loop)

Full write-up: **`iterative_eval_workflow.md`** (same folder)—system prompt → eval dataset → reference answers → runs → grader (e.g. 0–10) → average → iterate on the **same** suite; pitfalls include **eval local maxima** and **overfeeding** the system prompt. **Concrete run loop:** **`core_evaluation_pipeline.md`** (`run_prompt` → `run_test_case` → `run_eval`). **Graders:** **`prompt_evaluation_graders.md`**; **code/syntax side:** **`code_based_grading.md`**. Course skeleton: **draft prompt → build dataset → run model → grade → iterate** (`Claude/course_video_reference.md`, *Typical Eval Workflow*).

---

## Interview / task angles

- **Engineering vs eval** — Engineer for clarity and constraints; evaluate on representative tasks; iterate from failures and metrics.
- **Iterative pipeline** — See **`iterative_eval_workflow.md`** (grader loop, averages, prompt bloat / local maxima).
- **Production evaluation** — Manual review + structured cases + automated scoring; track regressions.
- **When is a pipeline worth it?** High impact, frequent changes, or costly failures (support, wrong actions, compliance).
- **Minimal eval** — Small golden set, clear pass/fail rules, rerun on prompt changes.

---

## Quick recap

| Topic | One line |
|--------|----------|
| **Prompt engineering** | *How* to write (few-shot, XML/structure, practices) so Claude understands intent and format. |
| **Prompt evaluation** | *Whether* it works—expected answers, version comparisons, suites, pipelines. |
| **Workflow** | Smoke → many runs / corner cases → optional **eval pipeline** for sustained confidence. |
| **Iterative eval loop** | See **`iterative_eval_workflow.md`** — fixed dataset + grader + average; same-suite regression; avoid **prompt bloat** / **local maxima**. |

Use **both** craft and measurement; pick eval depth by **risk**, **change frequency**, and **cost of wrong answers**.
