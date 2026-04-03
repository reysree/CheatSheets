# Iterative eval workflow (dataset + grader loop)

Study notes on a **typical pipeline** for building an **eval dataset** and **iterating the system prompt**: probe → fixed questions → expected answers → runs → grader scores → aggregate → repeat. Aligns with the *Typical Eval Workflow* in `Claude/course_video_reference.md` (draft prompt → dataset → instantiate → run → grade → iterate).

**Related:** `prompt_engineering_and_evaluation.md` (Parts 1–2: engineering vs eval, smoke → corner cases → pipeline). **Dataset generation:** `eval_dataset_generation_example.md`. **Running the pipeline:** `core_evaluation_pipeline.md`. **Graders (code / model / human):** `prompt_evaluation_graders.md`; **syntax + hybrid scoring:** `code_based_grading.md`.

---

## Overview

You start with a **use-case-driven system prompt**, observe behavior on **several question types**, then lock a **repeatable eval suite**. A **grader layer** scores model outputs against **expected** answers (e.g. **0–10**). You **average** scores to compare prompt versions on the **same** dataset. Improving the average suggests a better prompt **on that eval**—but **overfeeding** the system prompt can create a **local maximum** that breaks outside the suite.

---

## Core ideas

| Term | Meaning |
|------|--------|
| **Eval dataset** | Stable set of test questions (and often reference answers) reused across prompt versions for regression. |
| **Expected / gold answer** | What “good” looks like for each case—the grader’s target. |
| **Grader** | Code, model-as-judge, human, or hybrid; outputs a score or pass/fail per case. |
| **Aggregate metric** | Often **mean** (or weighted) score over cases—headline number for one prompt on one suite. |

---

## How it works (step-by-step)

| Step | What you do |
|------|-------------|
| **1. Initial system prompt** | Write a first system prompt for the **use case**, then **probe** it on **several kinds of questions** (formats, edge cases, domains you care about). Learn how the stack behaves before you trust aggregate metrics. |
| **2. Eval dataset** | Build test questions—ideally one suite that **covers** the types and failure modes you care about (single JSON/array or multiple files; **consistent reuse** for regression matters most). |
| **3. Reference answers** | For each case, define **expected** answers—often by running the model once and curating, or mixing human + model labels. The grader compares **actual** vs **expected**. |
| **4. Batch runs** | Feed the **same questions** through the API with the **current** system prompt (plus tools/RAG if applicable). Collect answers. |
| **5. Grader layer** | Score each response **against** the expected answer—numeric scale (e.g. **0–10**), rubric, or hybrid (code checks + model judge). **Code**, **model**, and **human** graders each have trade-offs; model judges can be **inconsistent**—monitor that. |
| **6. Aggregate** | **Average** (or weighted average) scores across cases → one number for “how good is this prompt **on this dataset**.” |
| **7. Iterate** | **Tweak** the system prompt (small, hypothesis-driven changes). Re-run the **identical** dataset and grader. If the **average rises**, the new prompt is **better on that eval**; if it drops, revert or revise. **Repeat** until gains flatten or diminishing returns. |

**Rule of thumb:** Compare prompt versions on the **same** fixed eval set. Changing both the questions and the prompt in one step makes it unclear what improved.

---

## Trade-offs

- **Fixed suite + same grader** — Enables fair **before/after** comparisons; cheap to automate once built.
- **Average score** — Easy to track; also inspect **per-case** failures so you fix real patterns, not only the mean.
- **Cost** — Dataset curation, grader calls (especially LLM judges), and maintenance as the product evolves.

---

## Pitfalls

- **Local maxima on the eval set** — The prompt **overfits** the suite: metrics look great, behavior **regresses** on real users or paraphrased inputs.
- **Overfeeding context in the system prompt** — Too many examples, edge-case rules, or bloated instructions can **inflate** eval scores while hurting **generalization**; it can feel “optimal” until simpler or shifted queries **break**.
- **Moving targets** — Editing the eval set and the prompt together without a hypothesis confounds measurement.
- **Grader blind spots** — A bad or noisy grader optimizes the wrong thing; validate graders on a **human** sample periodically.

**Mitigations:** Hold a **separate** stress or production-like set; add **new** cases over time; keep instructions **minimal** unless each addition shows a clear, measured win.

---

## Interview angles

- **Why use a grader?** — Scales scoring across many cases; supports automated regression on prompt changes.
- **Why average?** — Summarizes suite performance; combine with **failure analysis** on low-scoring cases.
- **What does “better prompt” mean here?** — **Higher average on the same dataset and grader**—not proof of universal quality; watch overfitting and grader bias.
- **Risk of stuffing the system prompt?** — **Local optimum** on the eval, **worse** generalization, surprise failures in production-like inputs.

---

## Quick recap

- **Flow:** System prompt v0 → diverse probe → eval dataset + reference answers → run → grader → **average** → tweak prompt → rerun **same** suite.
- **Success signal:** Average score **up** on **fixed** data + grader (with caveats).
- **Watch:** Eval **overfitting**, **prompt bloat**, **grader** quality, and changing the suite and prompt **together** without control.
