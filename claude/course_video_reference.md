# Claude video course — consolidated reference

Standalone study material aligned with the course notes. Pair with per-topic files under **`Claude/Accessing-Claude-with-API/`** (e.g. `api_usage.md`, `streaming_responses.md`).

---

## Overview of Claude Models

- **Opus**: Highest capability for complex, multi-step reasoning; higher cost and latency.
- **Sonnet**: Balanced intelligence, speed, cost; strong coding and edits; default for many apps.
- **Haiku**: Fastest, cost-efficient; not positioned like Opus/Sonnet for deep reasoning; good for realtime and high volume.
- **Selection**: Intelligence → Opus; speed → Haiku; balanced → Sonnet.
- **Pattern**: Mix models in one app by subtask rather than one model everywhere.
- **Shared**: Text, code, images; differences are optimization focus.

## Accessing the API

**Flow (client → server → Anthropic → client)**

1. Client sends text to **your server** (do not call Anthropic from the browser with a secret key).
2. Server calls Anthropic (SDK or HTTP): **API key**, **model**, **messages**, **max_tokens**.
3. Generation stages: **tokenization** → **embedding** → **contextualization** → **generation** (next-token probabilities + sampling).
4. Stops at **max_tokens** or end-of-sequence.
5. Response includes text, **usage**, **stop_reason**; server returns to client.

**Terms**: Token; embedding; contextualization; max_tokens; stop_reason.

## Making a Request

- Setup: `pip install anthropic python-dotenv`; `.env` with `ANTHROPIC_API_KEY`; `load_dotenv`; `Anthropic()` client; pick model (e.g. Sonnet).
- Call: `client.messages.create(model=..., max_tokens=..., messages=[...])`.
- **max_tokens**: safety cap on generation length, not a “target length.”
- Messages: `{"role": "user", "content": "..."}`; assistant messages for history.
- Text extraction: e.g. `response.content[0].text`.

## Multi-Turn Conversations

- API **does not store** chat history; each call is stateless.
- You must **append** user and assistant turns and **send full history** on each request.
- Helpers: add user message, add assistant message, `chat(messages)` wrapping the API.
- Without history, follow-ups lack context.

## System Prompts

- Pass **`system`** string to `create` to set role/behavior (tone, style), not to smuggle facts.
- Often: first line = role; then behavioral rules.
- Implementation detail: build `params`, add `system` only if non-None, `**params`.

## Temperature

- **0–1** scales randomness of next-token choice.
- **0**: nearly deterministic (highest-probability token).
- **Higher**: more weight on lower-probability tokens; more variety.
- Low: extraction, consistency; high: brainstorming, creative writing.
- Higher T increases *chance* of variation, not a guarantee.

## Response Streaming

- Problem: long waits without feedback.
- **stream=True** or `client.messages.stream()`; events include `message_start`, `content_block_start`, **`content_block_delta`** (text chunks), `content_block_stop`, `message_stop`.
- **`text_stream`** simplifies consuming text; **`get_final_message()`** for full message to persist.

## Controlling Model Output

**Assistant prefill**

- Put a partial **assistant** message at the end of `messages`; model continues from the **exact** end of that string; you may need to stitch prefill + completion.

**Stop sequences**

- Stop when a given substring would be emitted; that substring is **not** included. Refine strings (e.g. `", five"`) for cleaner cuts.

## Structured Data (prefill + stop)

- Problem: model adds chatter/markdown around JSON/code.
- Pattern: user asks for structure → assistant prefill opening delimiter (e.g. ` ```json `) → **stop sequence** closing delimiter (e.g. ` ``` `) → parse clean payload.

## Prompt Evaluation

- Prompt engineering = writing prompts; **evaluation** = systematic measurement (datasets, scores, iteration).
- Avoid “try twice then ship”; use datasets and metrics where possible.

## Typical Eval Workflow

1. Draft prompt  
2. Build **eval dataset** (small or large; hand-written or generated)  
3. Instantiate prompt per case  
4. Run model  
5. **Grade** outputs (code, model, or human graders)  
6. Iterate and compare versions  

## Generating Test Datasets

- JSON array of cases (e.g. `{ "task": "..." }`).
- Can use Claude (e.g. Haiku) with structured prefill/stop to emit JSON; save as `dataset.json`.

## Running the Eval

- **`run_prompt`**: merge case into prompt → API → output  
- **`run_test_case`**: run + grade → summary  
- **`run_eval`**: loop dataset  

Improve grading beyond placeholder scores.

## Model-Based Grading

- **Code graders**: length, keywords, syntax, etc.
- **Model graders**: separate LLM call with criteria + reasoning + score (JSON via prefill/stop).
- **Human**: flexible, expensive.
- Average scores across cases; watch grader inconsistency.

## Code-Based Grading

- Validate JSON / AST / regex compilation; binary or scaled scores.
- Dataset marks **expected format**; prompt insists on raw output; combine semantic + syntax scores (e.g. average).

## Prompt Engineering (module pattern)

- Start weak prompt → apply techniques stepwise → measure after each change.
- Example domain: meal plans with structured inputs; concurrent eval runs; HTML or other reports.

## Being Clear and Direct

- First line: **imperative** + task + output spec.  
- Strong baseline for score gains.

## Being Specific

- **Attributes** (length, format, sections) and/or **steps** (reasoning recipe). Often combine; big score jumps possible.

## Structure with XML Tags

- Wrap injected chunks in tags (`<sales_records>`, `<my_code>`) so the model knows boundaries.

## Providing Examples

- One-shot / multi-shot in tags; place after instructions; use eval winners as templates; explain why examples are good.

## Introducing Tool Use

- Claude may request **tool_use**; your server runs tools; you return **tool_result** blocks; loop until final answer.
- Enables live data (weather, DB, etc.).

## Project Overview (reminders)

- Gaps: “now,” duration math, persisting reminders → **datetime**, **add duration**, **set reminder** tools.

## Tool Functions

- Plain Python; clear names; validate inputs; meaningful errors so the model can retry.

## Tool Schemas

- JSON Schema: `name`, rich `description`, `input_schema`.
- Can draft schema with Claude + docs; wrap with `ToolParam` where needed.

## Handling Message Blocks

- With tools, **content** is multi-block (text + `tool_use`).
- Append **full** assistant `content` to history, not text-only.

## Sending Tool Results

- **`tool_result`**: `tool_use_id` matches request; `content` string (often JSON); `is_error` flag.
- Tool results go in a **user** turn; resend **tools** definitions as required by your stack.

## Multi-Turn Conversations with Tools

- Chain: ask → tool_use → execute → tool_result → … → final text.
- **`run_conversation`**: loop until `stop_reason` is not `tool_use` (pattern).

## Implementing Multiple Turns

- Inspect `stop_reason == "tool_use"`; **`run_tools`** maps blocks to functions; collect **`tool_result`** blocks; errors via `is_error`.

## Using Multiple Tools

- Extend **tools** list, **router** (`run_tool`), and implementations.

## The Batch Tool

- Single tool accepting a list of invocations to parallelize sub-calls the model might otherwise do serially.

## Tools for Structured Data

- Define tool whose **input** matches desired JSON shape; use **`tool_choice`** `{ "type": "tool", "name": "..." }` to force a call; read structured args from `tool_use`.

## Fine-Grained Tool Calling (streaming)

- Streaming may include **`input_json_delta`** (partial JSON).
- **Fine-grained** mode: less server-side JSON validation, faster partial args, client must tolerate invalid partial JSON.

## The Text Edit Tool

- Built-in **text editor** tool type; you implement filesystem/editor behavior; stub schema may expand per model version.

## The Web Search Tool

- Built-in web search (`web_search_20250305` / `web_search`); optional `max_uses`, `allowed_domains`; citations and result blocks in the response.

## Introducing RAG

- Large docs don’t fit or work well as one prompt → **chunk**, **retrieve** relevant pieces, then answer.

## Text Chunking Strategies

- By size (with overlap), by structure (headers), or semantic chunking; tradeoffs by doc type.

## Text Embeddings

- Embeddings map text to vectors; **semantic search** retrieves chunks by similarity (e.g. Voyage + vector store).

## The Full RAG Flow

1. Chunk  
2. Embed chunks  
3. Normalize (often automatic)  
4. Store in vector DB  
5. Embed query  
6. Similarity (e.g. cosine)  
7. Prompt with retrieved chunks  

## Implementing the RAG Flow

- Chunk → embed → `add` with metadata → query embed → `search` → build prompt.

## BM25 Lexical Search

- Keyword statistics; good for exact terms; often **hybrid** with vectors.

## A Multi-Index RAG Pipeline

- Parallel indexes + **Reciprocal Rank Fusion** (RRF) to merge ranks.

## Reranking Results

- LLM reorders candidates for relevance; extra latency; helps nuanced intent.

## Contextual Retrieval

- Prepend LLM-generated **context** to each chunk before indexing so chunks aren’t orphaned.

## Extended Thinking

- Separate thinking block + signed; budget ≥ 1024 tokens typical; **`max_tokens` > thinking budget**; costs for thinking tokens.

## Image Support

- Image blocks (base64 or URL); limits on count/size; tokens depend on resolution; **prompting** matters for quality.

## PDF Support

- `document` + `application/pdf`; similar pipeline to images.

## Citations

- Enable citations in request; locations in PDF or plain text; good for verifiable UI.

## Prompt Caching

- Reuses heavy prefix processing; **ephemeral** cache; **breakpoints** on tools/system/messages; max **4** breakpoints; **≥1024 tokens** per cached segment; order **tools → system → messages**; 1-hour TTL (per course).

## Prompt Caching in Action

- Put `cache_control: { "type": "ephemeral" }` on last tool block / system block as patterns; watch **cache_creation** vs **cache_read** tokens.

## Code Execution and the Files API

- Upload files → file IDs; code execution in isolated containers (no network); combine for analysis pipelines.

## Introducing MCP

- **MCP**: standard way to expose tools/resources/prompts; reduces bespoke integration code.

## MCP Clients

- **list_tools**, **call_tool**; stdio/HTTP/WebSocket transports.

## Project Setup (MCP tutorial)

- In-memory docs; both server and client in one learning repo.

## Defining Tools with MCP

- Python SDK `@mcp.tool`, typed params, `Field` descriptions.

## The Server Inspector

- `mcp dev server.py` for interactive testing.

## Implementing a Client

- Session wrapper: `list_tools`, `call_tool`.

## Defining Resources

- `@mcp.resource` URIs; templated URIs with parameters.

## Accessing Resources

- `read_resource`; parse JSON vs text by MIME type.

## Defining Prompts

- `@mcp.prompt` returns message templates for clients.

## Prompts in the Client

- `list_prompts`, `get_prompt(name, arguments)`.

## Anthropic Apps

- **Claude Code**, **Computer Use** as reference agentic apps.

## Claude Code Setup

- Node install; `claude` CLI; docs for full setup; MCP via `claude mcp add`.

## Claude Code in Action

- `init` → `claude.md`; memories; #notes; plan-then-implement; TDD style workflows.

## Enhancements with MCP Servers

- Modular capabilities (Sentry, Jira, custom tools).

## Parallelizing Claude Code

- **Git worktrees** per parallel agent; custom commands; merge back.

## Automated Debugging

- Scheduled jobs + logs + PRs from proposed fixes (example pattern).

## Computer Use

- Screenshot + UI actions in isolated environment; reference Docker implementation.

## How Computer Use Works

- Same tool loop: tool_use → your environment executes actions → tool_result.

## Agents and Workflows

- **Workflows**: known steps, reliable testing.  
- **Agents**: unknown steps, tool planning.  
- Use workflows when steps are clear; agents when exploration is required.

## Parallelization Workflows

- Split subtasks → parallel calls → aggregate.

## Chaining Workflows

- Sequential specialized calls; helps heavy constraint lists.

## Routing Workflows

- Classify input → route to specialized prompts/tools.

## Agents and Tools

- Prefer **small sets of composable tools** over dozens of hyper-specific ones.

## Environment Inspection

- Read state after actions (screenshots, file reads) for robust loops.

## Workflows vs Agents

- Workflows: predictable, easier to test. Agents: flexible, harder to test; prefer reliability first.

---

## Quick recap

Models differ by cost/capability; API is stateless; control output via system prompt, temperature, prefill/stop, streaming, tools, and structured extraction patterns. Production stacks add evals, RAG (chunk → embed → retrieve → optional BM25 + RRF + rerank), caching, MCP for integrations, and clear workflow vs agent boundaries.
