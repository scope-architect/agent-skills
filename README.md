# Scope Architect agent skills

Install into your repo with the vendor-neutral skills CLI:

```
npx skills add scope-architect/agent-skills
```

| Skill | What it teaches your agent |
|---|---|
| `scope-architect` | The task workflow over the hosted MCP: load the blueprint, `in_progress` before code, verify acceptance criteria, write status and comments back, keep project memory. |
| `agent-model-choice` | Model selection, cost estimation, and tool recommendations. Works against the public MCP endpoint, no account needed. |
| `autopilot` | The unattended loop: next ready task → implement → verify → done → repeat, with the scope as the only loop state and clear stop conditions. |

Scope Architect's MCP is hosted. Connect from Settings → MCP at https://app.scopearchitect.com. Docs: https://scopearchitect.com/docs
