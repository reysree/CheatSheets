# Learning notes convention

Study notes for **revision**, **interviews**, and **tasks** live in a **topic folder** with **descriptive snake_case** markdown files.

## Layout

| Piece | Rule |
|--------|------|
| **Topic folder** | Repo root, **PascalCase** (e.g. `Claude/`, `Kafka/`). |
| **Files** | **snake_case** `.md` names that say what’s inside, e.g. `overview_claude_models.md`, `streaming_api.md`. |

## How to use with Cursor

Say you’re **learning about** a topic (or studying / revising). The project rule adds or updates notes under the right folder with clear structure.

For **Claude** (API, prompts, tools, RAG, MCP), say you want to **learn**, **revise**, or be **quizzed** / **cross-examined**: the assistant uses **`Claude/course_video_reference.md`** plus topic notes under **`Claude/Accessing-Claude-with-API/`** and elsewhere in **`Claude/`** (see `.cursor/rules/claude-learning-tutor.mdc`).

## Suggested sections

1. **Overview** — What this is and why it matters.  
2. **Core ideas** — Terms and mental model.  
3. **How it works / usage** — Steps, examples.  
4. **Trade-offs** — Limits, comparisons.  
5. **Pitfalls** — Common mistakes.  
6. **Interview / task angles** — Short Q&A style notes.  
7. **Quick recap** — Bullets for a fast skim.
