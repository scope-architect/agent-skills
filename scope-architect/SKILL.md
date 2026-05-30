---
name: scope-architect
description: Load and navigate your Scope Architect project blueprint, update task status, append notes, and stay on-scope during implementation.
activationKeywords:
  - scope
  - tasks
  - what to work on
  - what's done
  - what's left
  - project progress
  - blueprint
  - deliverables
  - scope architect
---

# Scope Architect — Agent Workflow Guide

You have a live connection to Scope Architect via MCP. The blueprint in Scope Architect is the source of truth for what to build, what's done, and what's left.

## Session Start

1. Call `list_projects` to see available projects
2. Call `get_project(project_id, agent="claude_code")` for the active project
3. Read the full blueprint before touching any code
4. Call `read_comments` on any items with open questions or @mentions

## Naming Map

| UI Term | Tool param | Notes |
|---------|-----------|-------|
| Module | `module_name` | Top-level grouping (e.g. "Authentication") |
| Deliverable | `deliverable_name` | Feature-level item inside a module |
| Task | `task_desc` | Use the `id` field from `get_project` — shorter and matches exactly |

## Task Workflow

```
get_project → pick task → update_task_status(in_progress)
→ implement → check_acceptance_criterion for each AC
→ update_task_status(done)
→ append_task_note(decisions made, edge cases, anything noteworthy)
```

Or use `get_next_ready_task` to skip straight to the next approved work item.

## Handling Blockers — Never Delete

Do not use any delete operation. If a task can't be completed:
1. `update_task_status(..., status="blocked")`
2. `append_task_note(...)` — explain exactly what's blocking it

The team reviews blocked tasks in the UI and decides how to proceed.

## When to Add or Edit Scope

- **`add_task`** — discovered work not in the original scope (new requirement, hidden dependency)
- **`add_feature`** — new capability that belongs inside an existing module
- **`add_module`** — entirely new functional area that doesn't fit anywhere existing
- **`edit_task`** / **`update_feature`** / **`update_module`** — description needs clarification
- **`add_acceptance_criteria`** — implementation revealed additional conditions for done
- **`set_dependencies`** — task ordering must be enforced

Always call `get_project` first to check what already exists before adding anything.

## Stay On Scope

Do not build features, refactor code, or make UX changes that aren't in the blueprint. If something's missing, add it to the scope first and flag it to the user before building.

## Quick Reference

| Goal | Tool |
|------|------|
| See all projects | `list_projects` |
| Load blueprint | `get_project(project_id, agent="claude_code")` |
| Next approved task | `get_next_ready_task(project_id)` |
| Start a task | `update_task_status(..., status="in_progress")` |
| Finish a task | `update_task_status(..., status="done")` |
| Can't proceed | `update_task_status(..., status="blocked")` + `append_task_note` |
| Log what you did | `append_task_note(..., note="...")` |
| Tick off AC | `check_acceptance_criterion(..., criterion_index=0, done=true)` |
| Add AC | `add_acceptance_criteria(..., criteria=[...])` |
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
