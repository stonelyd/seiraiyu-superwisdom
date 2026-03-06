---
name: teams
description: Orchestrate Claude Code agent teams. When to use teams vs single agent vs subagent.
allowed-tools: TeamCreate TeamDelete TaskCreate TaskUpdate TaskList TaskGet TaskStop TaskOutput SendMessage
---

# Teams

Prefer agent teams (`TeamCreate`) for parallel execution across independent tasks. Fall back to `Agent` subagents only for quick single-purpose dispatches that only report results back.

## Teams vs subagents

| | Subagents | Agent teams |
|---|---|---|
| **Context** | Own window; results return to caller | Own window; fully independent |
| **Communication** | Report back to main agent only | Teammates message each other directly |
| **Coordination** | Main agent manages all work | Shared task list with self-coordination |
| **Best for** | Focused tasks where only the result matters | Complex work requiring discussion and collaboration |

## When to use teams

- 3+ independent tasks that can run in parallel
- Work where teammates benefit from sharing findings and challenging each other
- Multi-file changes where each teammate owns different files
- Research, review, and debugging with competing hypotheses

## When to use single agents instead

- Tasks with shared state or sequential dependencies → serialize them
- Quick single-purpose work → use `Agent` subagent
- Tasks touching the same files → run sequentially to avoid conflicts
- Routine tasks where coordination overhead exceeds the benefit

## Patterns

**Parallel workers:** `TeamCreate` → assign independent tasks → each teammate owns separate files → lead reviews results

**Research swarm:** Multiple teammates investigate different aspects → share findings and challenge each other → lead synthesizes

**Competing hypotheses:** Teammates test different theories in parallel, actively trying to disprove each other → surviving theory is likely correct

**Pipeline:** Sequential handoff between specialized teammates: design → implement → test → review

## Sizing

- Start with 3–5 teammates for most workflows
- Aim for 5–6 tasks per teammate to keep everyone productive
- Scale up only when the work genuinely benefits from more parallelism
- Three focused teammates often outperform five scattered ones

## Coordination

- Shared task list is the single source of truth
- Lead creates tasks via `TaskCreate`, assigns via `TaskUpdate`
- Tasks support dependencies — blocked tasks auto-unblock when dependencies complete
- Teammates self-claim the next unassigned, unblocked task after finishing
- Use `SendMessage` for direct messages; broadcast sparingly (costs scale with team size)

## Context at spawn

Teammates load project context (CLAUDE.md, MCP servers, skills) but do not inherit the lead's conversation history. Include task-specific details in the spawn prompt so teammates have enough context to work independently.

## Plan approval

For complex or risky tasks, require teammates to plan before implementing. The teammate works in read-only plan mode until the lead approves. If rejected, the teammate revises and resubmits.

## Cleanup

- Lead must run cleanup — teammates should not
- Shut down all teammates before cleaning up the team
- One team per session — clean up the current team before starting a new one
- Teammates cannot spawn their own teams (no nesting)
