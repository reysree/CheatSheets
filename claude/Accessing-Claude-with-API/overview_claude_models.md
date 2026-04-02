# Overview — Claude models

_Add notes as you learn. Add more files in `Claude/Accessing-Claude-with-API/` (or sibling topic folders under `Claude/`) for other subtopics (e.g. `prompting_claude.md`)._

## Overview

**Claude** is Anthropic’s family of language models. Choosing the **right model for each task** matters: you balance **intelligence**, **cost**, and **speed** so work gets done without wasting capability or money.

### Model family (when to use which)

| Model | Intelligence | Cost | Speed / latency | Reasoning | Best use cases |
|--------|----------------|------|------------------|-----------|----------------|
| **Claude Opus** | Highest | Highest | Moderate (higher than Sonnet/Haiku) | Yes | Heavy research, R&D, long-running work, strategic planning — “front-load” deep thinking before execution. |
| **Claude Sonnet** | Strong (balanced) | Mid | Balanced | Yes | Default **all-rounder**: coding, docs, testing, analysis, automation, most day-to-day product work. |
| **Claude Haiku** | Lower than Opus/Sonnet | Lowest | Fastest | — (lighter tasks) | Quick answers, orchestration, small fixes when you already know the fix, translation, latency-sensitive or high-volume simple tasks. |

**Principle:** **Match model to use case.** Using Opus for trivial tasks wastes money; using Haiku for hard research risks quality. Always weigh **how hard** the task is, **how wrong** answers can be, and **how fast** you need a reply.

## Core ideas

- Three tiers: **Opus** (max capability), **Sonnet** (balance), **Haiku** (speed/cost).
- **Reasoning** is strongest on Opus and Sonnet; reserve Haiku for lighter or well-scoped work.

## How it works / usage

- Start from **Sonnet** for general coding and analysis unless you need Opus-level depth or Haiku-level speed.
- Use **Opus** when the bottleneck is judgment, synthesis, or long-horizon reasoning, not raw tokens per second.
- Use **Haiku** for routing, simple edits, and high-throughput low-risk steps.

## Trade-offs

| Aspect | Opus | Sonnet | Haiku |
|--------|------|--------|--------|
| Capability | Highest | Strong | Good for simple/fast paths |
| Cost | Highest | Middle | Lowest |
| Latency | Higher | Middle | Lowest |
| Default for “daily coding” | Overkill unless hard | **Yes** | Only if speed/cost dominates |

## Pitfalls

- Using the **most capable** model everywhere → unnecessary cost and latency.
- Using the **cheapest/fastest** model for hard reasoning → rework and errors.
- Forgetting **task risk**: high-stakes or ambiguous work may justify Opus or careful Sonnet prompting.

## Interview / task angles

- *“How do you pick a model?”* → Task difficulty, error cost, latency/cost budget, need for deep reasoning vs. quick completion.
- *“Where does each tier fit?”* → Opus = research/strategy/heavy; Sonnet = general engineering; Haiku = fast/cheap/simple.

## Quick recap

- **Opus**: smartest, expensive, slower; best for R&D, long tasks, strategic planning.
- **Sonnet**: balanced; best default for coding, docs, tests, analysis, automation.
- **Haiku**: fastest, cheapest; best for quick answers, orchestration, small known fixes, translation.
- Optimize on **intelligence + cost + speed** together, not one dimension alone.
- Wrong model = wasted performance, money, or time.

## References

- [Anthropic — Claude](https://www.anthropic.com/claude) (verify current model names, pricing, and capabilities; they change over time.)
