---
name: agent-model-choice
description: Choose an AI model for a coding-agent task, estimate what a build will cost, and pick libraries that agents work well with. Use when deciding which model to run something on, comparing model pricing or context windows, budgeting an agent build, or choosing a component library or tool for an AI-assisted project.
activationKeywords:
  - which model
  - model pricing
  - model cost
  - cheapest model
  - context window
  - how much will it cost
  - token cost
  - component library
  - ui library
  - what should I build this with
  - zero data retention
  - model comparison
---

# Choosing models and tools for agent work

Live data for all of this is available over MCP — `list_models`, `estimate_agent_cost`,
and `recommend_stack` need no authentication. **Query them rather than answering from
memory: prices change, and a confidently wrong price is worse than no answer.**

## The one fact that changes model choice

Coding agents read far more than they write. A typical task is roughly **ten times more
input tokens than output** — file reads, tool results, and the whole conversation replayed
on every turn.

So **input price dominates total cost**, and any comparison sorted on output price is
actively misleading. When two models are close, the one with cheaper input almost always
wins on a real agent workload.

## Picking a model

Ask what the task actually needs, in this order:

1. **Is the task ambiguous, or is the blast radius large?** Use a frontier model. This is
   the case where a cheap model is expensive — it produces plausible work that is wrong in
   ways you find out about later.
2. **Is the task well-scoped, with clear acceptance criteria?** Use a workhorse model.
   This is most work, and it is where a good scope pays for itself: a well-specified task
   runs fine on a model costing a fraction of the frontier tier.
3. **Is it mechanical and high-volume?** Classification, extraction, routing, summarising —
   use a cheap model. Verify the output cheaply rather than paying to get it right first time.
4. **Is it pure code generation against a settled plan?** A code-specialist model tends to
   produce tighter diffs and less commentary.
5. **Must the data stay on your infrastructure?** Filter to open-weight models, or use
   `list_models` with `zdr_only: true` for hosted models with zero data retention.

The honest summary: **most people default too expensive.** If the task is properly scoped,
a workhorse model does it. If it isn't properly scoped, no model saves you — fix the scope.

## Estimating cost

Use `estimate_agent_cost` with a task breakdown. It takes t-shirt sizes (XS–XL) and
complexity, and returns a cost band per model.

Two things to communicate whenever you report an estimate:

- **The ordering is reliable; the absolute number is a ballpark.** Real spend depends on how
  much of the repo the agent reads, how often it retries, and whether prompt caching hits.
- **A band, not a point.** Quoting a single figure implies a precision that does not exist.

If the user has a Scope Architect project, its real task breakdown is far better input than
a guess — sizes and complexity are already stored per task.

## Choosing libraries and tools

Use `recommend_stack`. The thing to understand about component libraries specifically:

**Copy-in beats imported, for agents.** A library like shadcn/ui copies component source
into the repo, so an agent can read and modify it. An imported component library is opaque —
the agent can only pass props it hopes exist. This is why agent-built UI converges on
copy-in libraries, and it is a structural advantage, not a taste preference.

For anything with a chat or agent interface, check AI Elements before rebuilding message
lists and streaming states by hand.

## What not to do

- **Don't quote prices from memory.** Call `list_models`. Model pricing moves.
- **Don't recommend the most expensive model by default.** It reads as caution; it is
  usually just waste.
- **Don't present an estimate as a quote.** Say it's an estimate, give the range.
- **Don't recommend a tool you cannot say something specific about.** `recommend_stack`
  returns an editorial note for each entry — use it, or say you're unsure.
