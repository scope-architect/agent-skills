---
name: autopilot
description: Run a Scope Architect project unattended. Pull the next ready task, implement it, verify the acceptance criteria, mark it done, repeat, until the queue is empty, a task is blocked, or the user's budget is reached.
---

# Scope Architect — Autopilot loop

Use this when the user says "run the module", "keep going until it's done",
"autopilot", "loop through the tasks", or otherwise leaves you to work
unattended for a long stretch. It works in your editor (Claude Code, Cursor,
Codex) and in a Scope Architect cloud run alike.

## The contract

- **The scope is the loop state.** Never keep a private task list. `get_next_ready_task` decides what is next and `update_task_status` records progress. If your context resets mid-loop, resume from the scope, not from memory.
- **One task per iteration.** Finish it fully (criteria verified, artifact linked, status `done`) before pulling the next one.
- **Stop conditions.** No ready task; a task is blocked and the block policy is `pause`; the budget the user set (tasks, wall-clock, or cost) is reached; or the same task has failed three times in a row.
- **Never bend the scope to make the loop pass.** If a task is wrong or impossible, mark it `blocked` with a `post_comment` explaining why and move on. Do not delete, rewrite, or silently skip tasks.
- **Stay on scope.** Nothing outside the blueprint gets built, refactored, or "cleaned up while you're there".

## Before the first iteration

1. `get_project(project_id)` — read the Context block and the **Agent block policy** (`multitask` or `pause`).
2. Confirm the budget if the user did not give one. Defaults: **5 tasks**, stop at the first blocked task under `pause`, one commit per task on a working branch, one PR at the end.
3. `post_comment` on the project: `Autopilot started — budget: <n> tasks, branch: <name>`.

## Each iteration

```
get_next_ready_task(project_id)
  → nothing ready?            stop; go to "Ending".
  → human_reply_pending?      update_task_status(task_id, in_progress), then continue that task.
update_task_status(task_id, in_progress)
read_comments(task_id)                       // @mentions, prior attempts, decisions
implement
run the project's own tests / lint / typecheck
check_acceptance_criterion(task_id, i, done=true)   // each criterion, only once verified
commit (or push the PR)  → link_artifact(task_id, url)
update_task_status(task_id, done)
post_comment(task_id, "Done: <one line>. Files: <paths>. Verified: <how>.")
```

If the task cannot be completed: `request_clarification` (preferred) or
`update_task_status(blocked)` + `post_comment` with an @mention. Under
`multitask`, continue with the next ready task. Under `pause`, stop the loop.

A task that fails three times is `blocked` with the three failure summaries in
the comment, and the loop moves on or stops per policy.

## Ending

1. `post_comment` on the project: tasks done, tasks blocked, artifacts, and what the human should review first.
2. `record_memory` for anything the next session must know (decisions taken, gotchas found). Not a diary, only durable facts.
3. Leave every task in a truthful status. Nothing `in_progress` when you stop.
