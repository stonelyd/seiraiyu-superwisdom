---
name: execute
description: Execute implementation plans using agent teams for parallel work and sequential agents for dependent tasks.
allowed-tools: TeamCreate TaskCreate TaskUpdate TaskList Agent Bash Read Edit Write Skill AskUserQuestion
---

# Execute

Load the plan, review it critically, execute using teams for independent work and sequential agents for dependent work.

## Before starting

1. Read the plan file completely
2. Review critically — raise concerns before writing any code
3. Create TodoWrite tracker for all tasks
4. Identify which tasks are independent vs dependent

## Execution

**Independent tasks (parallelize):**
- `TeamCreate` with worker agents in worktree isolation
- Assign tasks via `TaskUpdate`
- Workers execute, verify, commit, mark complete

**Dependent tasks (serialize):**
- Execute sequentially in current session
- Each task follows its verification steps before the next begins

## After all tasks

- Final full verification: test suite, linter, build
- Final code review of complete implementation
- Hand off to `git-flow` skill for branch completion

## When to escalate

- Blocker mid-batch → pause and use `AskUserQuestion` for guidance
- Plan has gaps → surface them via `AskUserQuestion` before continuing
- Verification fails repeatedly → switch to the `debug` skill
