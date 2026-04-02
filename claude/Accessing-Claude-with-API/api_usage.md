# Claude API — request flow and processing

Notes on how client apps reach Anthropic, what the API expects, what happens conceptually during generation, and what comes back—useful for **secure design**, **limits**, and **debugging**.

## Overview

Using Claude through the API is a **pipeline**: the **client** talks to **your server**, your server talks to **Anthropic** with credentials; Anthropic runs the model and returns a **structured response** (text, usage, **stop reason**); your server forwards the result to the app. You do **not** call Anthropic from untrusted front-end code if that would expose secrets (see stage 2).

A useful mental model splits the journey into **six stages** (below). Inside the model, people often describe **four phases** of processing—simplified, but good for intuition and for tracing failures.

## The six stages (end-to-end)


| Stage                                        | What happens                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Client → your server**                  | The browser, mobile app, or other client sends the user’s message to **your** backend (HTTP/API you control)—not directly to Anthropic with a secret in the client.                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| **2. Why a server (not client → Anthropic)** | Anthropic API calls **authenticate** with an **API key**, usually sent in a **header** (e.g. `x-api-key` / `Authorization`). That key is a **secret**: it proves who is billed and authorized. If the **frontend** called Anthropic over HTTP, the key would have to be **embedded in client code** or visible in every request—inspectable in devtools, bundles, or proxies. **Exposing the key is a serious security vulnerability** (account abuse, runaway cost, possible data/compromise risk). So you keep the key **only on a server** (or trusted serverless backend) and call Anthropic **from there**. |
| **3. Your server → Anthropic (SDK / HTTP)**  | Your backend uses the **Anthropic SDK** or HTTPS with the key stored in environment/secrets; Anthropic validates the request (key, model, required fields).                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| **4. Model processing**                      | Tokenization → numerical representations → contextual refinement → **autoregressive** generation (one chunk at a time until a stop condition).                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| **5. API response**                          | Structured payload: assistant message, **usage** (e.g. token counts), **stop reason** (why generation ended).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| **6. Server → client**                       | Your backend returns the generated text (and any metadata you choose) to the web/mobile client for the UI.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |


Stages 1–3 are “trust boundary + wire + contract”; stage 4 is “model”; 5–6 are “observability + product.”

## Essential request fields (Messages API)

Every call must carry enough information for Anthropic to **authenticate**, **select a model**, **know the conversation**, and **bound output length**. In practice you always include:


| Field            | Role                                                                                                                               |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| **API key**      | Identifies and authorizes your project/account with Anthropic (must be protected—never expose in client-side code for production). |
| **Model**        | Which Claude variant to run (e.g. Sonnet, Opus, Haiku—exact **IDs** are versioned strings in the API docs).                        |
| **Messages**     | Structured list of roles and content (user/assistant turns) carrying the user’s input and prior context.                           |
| `**max_tokens`** | Upper bound on how many **output** tokens the model may generate in that turn.                                                     |


Other parameters (temperature, top_p, stop sequences, system prompts, tools, etc.) tune behavior; the table above is the **minimum conceptual checklist** for “what must be there.”

## Inside the model (four phases — conceptual)

These steps describe **how people reason about** transformer language models; exact internals are proprietary and more detailed than this list.

1. **Tokenization** — Text is split into **tokens** (words, subwords, punctuation, etc.), the units the model scores.
2. **Embeddings / representations** — Tokens are mapped to **vectors** (long lists of numbers). Early layers combine token identity with position so similar or related meanings can be represented in geometry. (Pedagogically: “numbers that capture meaning and relationships”; technically it is **learned** representations, not a hand-written dictionary.)
3. **Contextualization** — Deeper layers **refine** representations using surrounding tokens so polysemy (“bank” river vs. money) and syntax are disambiguated in context.
4. **Generation** — The model predicts **likely next tokens** given everything so far. Sampling uses **probability** plus **controlled randomness** (temperature, top_p, etc.) so outputs stay fluent and varied, not always the single highest-probability token. Generation **iterates**: append token → run forward pass → sample next → repeat.

### When generation stops

Stopping is a **policy + model** decision, not magic. Common reasons:

- **Natural completion** — Model emits an **end-of-sequence (EOS)** style signal when the continuation looks complete (exact mechanism is implementation-specific).
- `**max_tokens` reached** — Hard cap on output length for that request.
- **Stop sequences** — Your app supplies phrases that halt generation when matched.
- **Other API/tooling limits** — Context window, safety, or tool-use flows can also bound behavior.

For debugging, always check **stop reason** and **usage** in the API response.

## API response (what you get back)

After generation finishes, the response is **structured**, typically including:

- **Message content** — The assistant’s generated text (and optionally structured blocks if using advanced features).
- **Usage** — Token counts (input/output) for billing and tuning.
- **Stop reason** — Why generation ended (e.g. `end_turn`, `max_tokens`, `stop_sequence`—**exact enum values** depend on API version; confirm in current docs).

Your server should log or propagate **stop reason + usage** when debugging latency, cost, or “why did it cut off?”

## Takeaways (design and operations)

- **Security / backend** — The API key belongs in **server-side** env/secrets only. **Never** put it in front-end code: browser calls to Anthropic would require sending the key in headers, exposing it to users and attackers.
- **Anthropic SDK** — Use official SDKs for correct auth, retries, and typing; reduces footguns vs. raw HTTP.
- **Token limits** — Set `max_tokens` (and context budget) to match the task: small for short replies, larger when you need long outputs—mis-set limits look like “truncated” answers.
- **Application-level stopping** — Combine API stop reasons with **your** logic: partial answers, max retries, user cancellation, or “good enough” matches for orchestration.
- **Debugging** — Trace the pipeline: bad prompt vs. token limit vs. model stop vs. network; **usage + stop reason** narrow it quickly.

## Pitfalls

- **Leaked API keys** — especially from **client-side** Anthropic calls (key in headers → visible); also repos, bundles, screenshots.
- **Max tokens too low** → answers stop mid-thought; looks like a model bug.
- **Ignoring stop reason** → you cannot explain truncations or tune limits.
- **Treating the four internal phases as literal source code** — helpful mental model; not a public specification of every tensor op.

## Interview / task angles

- *“What must every Claude API request include?”* → Auth (API key), model id, messages, and an output bound (`max_tokens`).
- *“Why did output stop?”* → Check stop reason, `max_tokens`, stop sequences, and natural EOS behavior.
- *“How do you secure API access?”* → Keys on server, env/secrets manager, no client embedding, audit and rotate.
- *“Why not call Anthropic directly from the client?”* → The API key must authenticate every request; in a browser it would leak (headers, bundles, devtools). Use a backend proxy that holds the key.

## Quick recap

- **Flow:** client → **your server** (no Anthropic secret in the browser) → Anthropic (SDK + key server-side) → model (tokenize → represent → contextualize → generate) → structured response → forward to client.
- **Must-haves:** API key, model, messages, `max_tokens`.
- **Observability:** usage + stop reason for cost and debugging.
- **Design:** secure keys, right limits, and clear stop semantics in app + API layers.

## References

- [Anthropic API documentation](https://docs.anthropic.com/) — request schemas, stop reasons, and model IDs change over time; verify against current docs.

