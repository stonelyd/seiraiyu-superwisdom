---
name: git-flow
description: Worktree setup and branch completion lifecycle. Start isolated work, finish with merge/PR/keep/discard.
---

# Git Flow

One skill for the full branch lifecycle: create an isolated worktree, do the work, finish with your choice of merge, PR, keep, or discard.

## Starting work — worktree setup

1. Check for existing worktree directory: `.worktrees` first, then `worktrees`
2. Check CLAUDE.md for directory preference
3. If unclear, use `AskUserQuestion` to ask
4. Verify `.gitignore` covers the worktree directory
5. Create: `git worktree add <path> -b <branch-name>`
6. Auto-detect and run setup (`npm install`, `pip install -e .`, etc.)
7. Run tests for a clean baseline

## Finishing work — branch completion

1. Verify all tests pass — stop if they don't
2. Use `AskUserQuestion` to present four options:
   - **Merge** to base branch locally
   - **Push and create PR** via `gh pr create`
   - **Keep** branch as-is for later
   - **Discard** — requires typing "discard" to confirm
3. Execute the chosen option
4. Clean up worktree for merge and discard options
