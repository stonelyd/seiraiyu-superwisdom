---
name: teams
description: Orchestrate Claude Code agent teams. When to use teams vs single agent vs subagent.
---

# Teams

Use `TeamCreate` and agent teams for parallel execution. Use `Agent` subagents for quick single-purpose dispatches.

## When to use teams

- 3+ independent tasks that can run in parallel
- Multi-file changes where agents won't conflict
- Large implementation plans with clear task boundaries

## When NOT to use teams

- Tasks with shared state or sequential dependencies
- Quick single-purpose work (use Agent subagent instead)
- Tasks touching the same files (merge conflict risk)

## Patterns

**Parallel workers:** `TeamCreate` → assign independent tasks → each agent works in worktree isolation → review merged results

**Research swarm:** Multiple agents investigate different aspects → report findings to lead → lead synthesizes

**Pipeline:** Sequential handoff between specialized agents: design → implement → test → review

## Coordination

- Task list is the single source of truth
- Lead assigns tasks via `TaskUpdate`
- Workers mark complete and check for next task via `TaskList`
- Use `SendMessage` DM for normal communication
- Broadcast only for blocking issues that affect everyone
