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

## Naming Map

The Scope Architect data model uses these terms — use exact names when calling tools:

| UI Term | Tool param | Notes |
|---------|-----------|-------|
| Module | `module_name` | Top-level grouping (e.g. "Brand Foundation") |
| Deliverable | `deliverable_name` | Feature-level item inside a module |
| Task | `task_desc` | Use the `id` field from `get_project` output (e.g. `"logo-discovery-moodboard"`) — it's shorter and matches exactly |

## Task Workflow

```
get_project → pick task → update_task_status(in_progress)
→ implement → update_task_status(done)
→ append_task_note(decisions made, edge cases, anything noteworthy)
```

Check acceptance criteria: `check_acceptance_criterion(criterion_index, done=true)` for each AC as you satisfy it.

## When to Add or Edit Tasks

- **`add_task`** — you discovered work not in the original scope (new requirement, hidden dependency)
- **`edit_task`** — the task description needs clarification or acceptance criteria need updating

## Stay On Scope

Do not build features, refactor code, or make UX changes that aren't represented in the blueprint. If you think something's missing, add a task first and flag it to the user before building.

## Quick Reference

| Goal | Tool |
|------|------|
| See all projects | `list_projects` |
| Load blueprint | `get_project(project_id, agent="claude_code")` |
| Start a task | `update_task_status(..., status="in_progress")` |
| Finish a task | `update_task_status(..., status="done")` |
| Log what you did | `append_task_note(..., note="...")` |
| Tick off AC | `check_acceptance_criterion(..., criterion_index=0, done=true)` |
| Scope gap found | `add_task(...)` |
| Rename project | `rename_project(project_id, name="New Name")` |
| Update status | `update_project_status(project_id, status="finalized")` |
| Recent changes | `recent_activity(project_id)` |
