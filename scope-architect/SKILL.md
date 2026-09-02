---
name: scope-architect
description: Load Scope Architect blueprints, update tasks, coordinate via comments, and persist project knowledge (memories, documentation) when decisions or reference material must outlive the session.
activationKeywords:
  - scope
  - scope architect
  - tasks
  - blueprint
  - deliverables
  - memory
  - documentation
  - context
  - ADR
  - runbook
  - project knowledge
  - what to work on
  - what's done
  - what's left
---

# Scope Architect — Agent Workflow Guide

You have a live connection to Scope Architect via MCP. The blueprint is the source of truth for what to build. **Project knowledge** (Memory + Documentation) is a **first-class capability** — use it when something must survive beyond this chat, not on every task.

## Project knowledge (prominent)

Three layers — don't confuse them:

| Layer | Who owns it | Agent tool | When to use |
|-------|-------------|------------|-------------|
| **Context** | Human (UI) | Read via `get_project` | Always-on rules — don't duplicate in memories |
| **Memory** | Agent + human | `record_memory` | Decision, preference, off-limits, what you tried — **next session** must know |
| **Documentation** | Agent + human | `write_document` | ADR, runbook, API notes — **long-form** reference |

| Goal | Tool |
|------|------|
| Save a durable fact (when needed) | `record_memory` |
| Find past decisions | `search_memories` / `list_memories` |
| Write an ADR or runbook (when needed) | `write_document` |
| Read a doc | `read_document` / `list_documents` |
| Promote fact to always-on Context | `promote_memory_to_context` |
| Archive obsolete memory | `archive_memory` |

**Use when needed or when the user asks** — e.g. architectural decision, non-obvious constraint, failed approach, runbook worth keeping. **Skip** for routine task work with no lasting lesson.

**Not substitutes:** `post_comment` = team-visible on a task. `implementationNotes` on `edit_task` = tech context on that task. Neither replaces memory/docs for project-wide continuity.

## Session Start

1. Call `list_projects` to see available projects
2. Call `get_project(project_id, agent="claude_code")` — includes knowledge briefing (Context, pinned docs, recent memories)
3. If the topic needs prior decisions: `search_memories` and/or `search_documents`, then `read_document` as needed
4. Read the full blueprint before touching any code
5. Call `read_comments` on the task (or project) for open questions and @mentions

## Communication — comments only

**All agent↔team communication goes through `post_comment`.** Mention people with `@username` (e.g. `@tie`).

| Do | Don't |
|----|--------|
| `post_comment` for blockers, questions, handoffs, summaries | Builder notes — **internal-only**; you cannot read or write them |
| `implementationNotes` via `edit_task` for durable tech context on **this task** | `append_task_note` — **removed**; use comments instead |
| `record_memory` when the **project** must remember (not every task) | Duplicating comment threads in memories |
| `write_document` for long-form reference (ADRs, runbooks) | Putting ADR-length content only in a memory |

## Stable IDs (always prefer UUIDs)

Every project, module, feature, and task has an `id` in `get_project` / `get_next_ready_task`. **Use these on write tools** — do not rely on name matching.

| Entity | Param | Where to get it |
|--------|--------|-----------------|
| Project | `project_id` | `list_projects` |
| Module | `module_id` | `get_project` → `modules[].id` |
| Feature / deliverable | `feature_id` | `get_project` → `modules[].features[].id` |
| Task | `task_id` | `get_project` or `get_next_ready_task` → `tasks[].id` |

Legacy name params (`module_name`, `deliverable_name`, `task_desc`) still work but are error-prone on large scopes.

## Naming Map (UI ↔ scope)

| UI Term | Short name | Detailed description |
|---------|------------|----------------------|
| Module | `modules[].name` | `modules[].technicalDesc` |
| Feature (Deliverable) | `features[].title` | `features[].technicalDesc` |
| Task | `tasks[].title` | `tasks[].technicalDesc` |

**When talking to humans, refer to items by their `title` (or module `name`) — never by raw ID.** IDs are tool arguments only. Keep `title` short (≤ ~80 chars); put detail in `technicalDesc` and discrete steps in `acceptanceCriteria`. On `add_task`/`add_feature`/`edit_task`, always set a short `title`.

## Task Workflow

```
get_project → read_comments(task) → pick task (note task_id)
→ update_task_status({ task_id, status: "in_progress" })
→ implement → check_acceptance_criterion({ task_id, ... }) for each AC
→ update_task_status({ task_id, status: "done" })
→ post_comment({ task_id, body: "Done: … @tie" })   // team-visible summary
→ [if needed] record_memory / write_document        // project knowledge — not every task
```

Or use `get_next_ready_task` — it returns `task_id` for the next item.

## Scope-first workflow (required for all sessions)

The blueprint is the work ledger. **Agent duration** in the UI counts **only time in `in_progress`** (not `blocked`, not `todo`). Multiple `in_progress` windows on one task are summed (e.g. in_progress → blocked → in_progress → done).

### Pre-work scope alignment (default — do this first)

Unless project **Context**, a user message, or another skill says to skip it:

1. **`get_project`** and review modules/tasks with the user (in chat or `post_comment` on the project).
2. Align on what is **in scope** vs deferred: `mark_agent_ready(false)` on irrelevant tasks, note exclusions in a project comment.
3. **`add_task` / `add_feature`** for missing work you already know you'll need — before implementation.
4. Confirm the **next task_id** and order, then start work.

This is how real teams use Scope Architect; skipping it hides product gaps.

### Two conscious modes (do not conflate)

| Mode | When | How to behave |
|------|------|----------------|
| **Real user test** | Exercising Scope Architect like a customer (e.g. Cloud VM official test) | MCP is the product. Full scope cycle. Assess gaps honestly with the user. |
| **Building Scope Architect** | Editing this repo's codebase (`src/`, `mcp/`, migrations) | Still use MCP when the work maps to a SA project; don't treat "we're building SA" as an excuse to skip tasks. |

When the user is testing the product with you, **prioritize their project scope** over drive-by app refactors unless they ask for both.

### Before writing code (every task)

1. Task must exist — **`add_task`** first if it doesn't.
2. **`update_task_status(..., in_progress)`** before the first file edit, shell command, or infra change.
3. Prefer one primary **`in_progress`** task at a time.

### Discovered work

| Situation | Action |
|-----------|--------|
| New migration, API route, or file area not in any task | `add_task` → `in_progress` → implement → `done` |
| Sub-step of a large task (>30 min) | `add_task` under same feature with clear title, or `add_acceptance_criteria` |
| Irrelevant preset / out-of-scope modules | `mark_agent_ready(false)`; do not implement |

### Human replied on a blocked task

1. `get_next_ready_task` may return `human_reply_pending` with their answer — task stays **`blocked`** until **you** change it.
2. **`update_task_status(..., in_progress)`** before resuming implementation (required; not automatic).
3. Do not mark **`done`** without an **`in_progress`** window for that stretch of work.

### When marking done

- `post_comment` with what shipped (paths, env vars, how to test).
- `edit_task` / `implementationNotes` for durable tech detail on that task.
- Do not mark a parent task done if child deliverables were never tracked as tasks.

**When to add knowledge after work:** non-obvious decision, new constraint, failed approach worth avoiding, or user asked you to remember/document something.

## Handling Blockers — Never Delete

Do not use any delete operation. If a task can't be completed:

### Default: async (multitask) — prefer this

1. `request_clarification({ project_id, task_id, question, context? })`  
   → @mention on the task, status **blocked**, human notified. **Do not guess.**
2. **Continue other work** — other tasks, scope updates, MCP housekeeping. Do **not** sit idle on this task.
3. **Check in before taking new work** (and at session start):  
   - `get_active_agent_runs({ project_id })`  
   - `get_next_ready_task({ project_id })` — if `human_reply_pending`, **finish that task first** using the returned answer  
   - `read_comments` on that `task_id` if you need full thread context
4. **`update_task_status(..., in_progress)`** then apply the human's answer and complete the blocked task — do **not** ask them to type "continue" in the terminal.

### Optional: sync (single-task only)

If you are **only** working this one task until the human answers:

1. `request_clarification` → then `await_human_reply({ project_id, task_id })` (blocks until they reply in SA Comments)  
2. Continue immediately with the returned body.

Fallback (legacy): `update_task_status(..., blocked)` + `post_comment` with @mention, then check in with `get_active_agent_runs` / `read_comments`.

## When to Add or Edit Scope

- **`add_task`** — discovered work not in the original scope (new requirement, hidden dependency)
- **`add_feature`** — new capability that belongs inside an existing module
- **`add_module`** — entirely new functional area that doesn't fit anywhere existing
- **`edit_task`** / **`update_feature`** / **`update_module`** — description needs clarification
- **`add_acceptance_criteria`** — implementation revealed additional conditions for done
- **`set_dependencies`** — task ordering must be enforced

Always call `get_project` first to check what already exists before adding anything.

## Autopilot (unattended loop)

When the user says "run the module", "keep going until done", or leaves you to work unattended, loop on the scope: `get_next_ready_task` → `in_progress` → implement → verify criteria → `link_artifact` → `done` → `post_comment` → repeat. The scope is the only loop state; never keep a private task list. Stop when nothing is ready, a task is blocked under a `pause` policy, the user's budget (tasks/time/cost) is reached, or the same task fails three times. Full rules live in the `autopilot` skill (installed alongside this one by `npx skills add scope-architect/agent-skills`).

## Stay On Scope

Do not build features, refactor code, or make UX changes that aren't in the blueprint. If something's missing, add it to the scope first and flag it to the user before building.

## Quick Reference

| Goal | Tool |
|------|------|
| See all projects | `list_projects` |
| Load blueprint + knowledge briefing | `get_project(project_id, agent="claude_code")` |
| Next approved task | `get_next_ready_task(project_id)` |
| Start a task | `update_task_status(project_id, task_id, status="in_progress")` |
| Finish a task | `update_task_status(project_id, task_id, status="done")` |
| Can't proceed | `update_task_status(..., blocked)` + `post_comment` with @mention |
| Summarise for team | `post_comment(...)` |
| Save durable project fact | `record_memory(project_id, kind, content)` |
| Search past decisions | `search_memories` / `list_memories` |
| Write ADR / runbook | `write_document(project_id, slug, title, content)` |
| Read project doc | `read_document` / `list_documents` |
| Tick off AC | `check_acceptance_criterion(project_id, task_id, criterion_index=0, done=true)` |
| Add AC | `add_acceptance_criteria(..., criteria=[...])` |
| Agent-visible tech notes (this task) | `edit_task(..., patch: { implementationNotes: "…" })` |
| Scope gap — task | `add_task(...)` |
| Scope gap — feature | `add_feature(...)` |
| Scope gap — module | `add_module(...)` |
| Set task order | `set_dependencies(..., depends_on=[...])` |
| Toggle agent queue | `mark_agent_ready(..., agent_ready=true/false)` |
| Ask the team | `post_comment(...)` with @username mention |
| Check messages | `read_comments(...)` with `mentions_of="claude_code"` |
| Rename project | `rename_project(project_id, name="New Name")` |
| Update status | `update_project_status(project_id, status="finalized")` |
| Recent changes | `recent_activity(project_id)` |
