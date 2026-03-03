# Superwisdom Plugin Implementation Plan

**Goal:** Implement all 10 skill files and README for the superwisdom plugin — a streamlined replacement for superpowers with professional tone, consolidated skills, and team-oriented execution.

**Architecture:** Each skill is a standalone `SKILL.md` with YAML frontmatter (name + description) and markdown body. Skills reference each other by name. The `boot` skill is the meta-skill loaded on every conversation. Word count targets: <150 words for boot, <200 for verify, <500 for all others.

**Tech Stack:** Markdown, YAML frontmatter, Claude Code plugin system (Seiraiyu Marketplace).

---

## Task 1: Create `boot` SKILL.md

**Files:**
- Create: `skills/boot/SKILL.md`

**Step 1: Write the skill file**

```markdown
---
name: boot
description: Meta-skill loaded on every conversation. Check for matching skills before starting any task.
---

# Boot

Check for matching skills before starting work. If one exists, use it.

## Before any task

1. Scan available skills
2. If a skill matches → load it with the Skill tool, announce ("I'm using [skill] to [action]"), follow it
3. If the skill has a checklist → create TodoWrite items for each step

If no skill matches, proceed normally.

## Checklists

When a skill contains a checklist, create a TodoWrite todo for each item. Don't work through checklists mentally — tracked items don't get skipped.

## Announcing

Before using a skill, say what you're doing:
- "I'm using the brainstorm skill to refine this idea into a design."
- "I'm using the tdd skill to implement this feature test-first."
```

**Step 2: Verify**

Run: `wc -w skills/boot/SKILL.md`
Expected: Under 150 words (excluding frontmatter)

**Step 3: Commit**

```bash
git add skills/boot/SKILL.md
git commit -m "feat: add boot skill — meta-skill for skill discovery"
```

---

## Task 2: Create `verify` SKILL.md

**Files:**
- Create: `skills/verify/SKILL.md`

**Step 1: Write the skill file**

```markdown
---
name: verify
description: Run verification before claiming work is done. Evidence before assertions.
---

# Verify

Before claiming work is complete, fixed, or passing — run the proof.

## Process

1. **Identify** — what command proves the claim?
2. **Run** — execute it fresh (not from cache or memory)
3. **Read** — full output, check exit code
4. **Confirm** — does the output support the claim?
5. **Then claim** — with the evidence

Applies to: test results, build output, bug fixes, linter output, deployments, agent reports.

## Don't say

- "Should work now"
- "Probably passes"
- "Seems fixed"
- "Tests were passing earlier"

Run it. Show the output.

## Reference

| Claim | Requires | Not sufficient |
|-------|----------|----------------|
| Tests pass | Test output: 0 failures | Previous run |
| Build succeeds | Build exit 0 | Linter passing |
| Bug fixed | Reproduction test passes | "Code looks right" |
| Linter clean | Linter output: 0 errors | Partial file check |
```

**Step 2: Verify**

Run: `wc -w skills/verify/SKILL.md`
Expected: Under 200 words (excluding frontmatter)

**Step 3: Commit**

```bash
git add skills/verify/SKILL.md
git commit -m "feat: add verify skill — evidence before assertions"
```

---

## Task 3: Create `tdd` SKILL.md

**Files:**
- Create: `skills/tdd/SKILL.md`

**Step 1: Write the skill file**

```markdown
---
name: tdd
description: Red-green-refactor discipline. Write the test first, watch it fail, write minimal code to pass.
---

# TDD

No production code without a failing test first.

## Red — Write failing test

- One minimal test for one behavior
- Clear name describing what's being tested
- Run it. Confirm it fails for the right reason.

## Green — Write minimal code

- Simplest code that makes the test pass
- Don't add features, don't refactor, don't anticipate
- Run tests. Confirm all pass.

## Refactor — Clean up

- Remove duplication, improve names, extract patterns
- Only after green
- Run tests. Confirm all still pass.

Repeat for the next behavior.

## Anti-patterns

- Testing mock behavior instead of real code through mocks
- Adding test-only methods to production code
- Mocking without understanding what you're replacing
- Writing production code before the test
- Test passes on first run — you didn't verify it catches failures

## Async timing

- Don't use `setTimeout` or `sleep` in tests
- Use condition-based waiting: `waitFor(() => condition)` or poll every 10ms with a 5s timeout
- Wait for state first, then optionally add a documented delay — never reverse the order

## Watch for

- "Too simple to need a test" → write it anyway, takes 30 seconds
- "I'll write the test after" → test-after proves nothing about failure detection
- "Just this once" → discipline only works if it's every time
```

**Step 2: Verify**

Run: `wc -w skills/tdd/SKILL.md`
Expected: Under 500 words (excluding frontmatter)

**Step 3: Commit**

```bash
git add skills/tdd/SKILL.md
git commit -m "feat: add tdd skill — red-green-refactor with anti-patterns and async timing"
```

---

## Task 4: Create `debug` SKILL.md

**Files:**
- Create: `skills/debug/SKILL.md`

**Step 1: Write the skill file**

```markdown
---
name: debug
description: Investigate, trace root cause, fix, harden. Understand before fixing.
---

# Debug

Understand the bug before attempting fixes. If 3+ fixes fail, stop and question the architecture.

## Phase 1 — Investigate

- Read error messages completely
- Reproduce consistently
- Check recent changes: `git log`, `git diff`
- Log data at component boundaries

## Phase 2 — Trace root cause

- Start at the symptom, trace backward through the call stack
- At each level: "What called this? Where did this data come from?"
- If manual tracing stalls, add instrumentation (`console.error` with stack traces) at key boundaries
- Find where the bad data originates — that's your root cause

## Phase 3 — Fix

- Form a single hypothesis — write it down
- Make the smallest possible change
- Write a failing test that reproduces the bug
- Implement the fix, verify the test passes
- If the fix doesn't work: return to Phase 1, don't stack another guess

## Phase 4 — Harden

- Validate at every layer the data passes through
- Entry point validation → business logic checks → environment guards
- Make the bug structurally impossible, not just patched

## Stop conditions

- 3+ fix attempts failed → question the architecture, not the fix
- Each fix reveals a new problem elsewhere → you're treating symptoms
- "Quick fix now, investigate later" → investigate now
```

**Step 2: Verify**

Run: `wc -w skills/debug/SKILL.md`
Expected: Under 500 words (excluding frontmatter)

**Step 3: Commit**

```bash
git add skills/debug/SKILL.md
git commit -m "feat: add debug skill — four-phase debugging workflow"
```

---

## Task 5: Create `review` SKILL.md

**Files:**
- Create: `skills/review/SKILL.md`

**Step 1: Write the skill file**

```markdown
---
name: review
description: Request and respond to code reviews. Verify suggestions before implementing.
---

# Review

Two modes: requesting a review and responding to review feedback.

## Requesting review

1. Get the git diff range (SHAs or branch comparison)
2. Dispatch `superpowers:code-reviewer` agent with:
   - What was implemented
   - Plan or requirements reference
   - Diff range
3. Reviewer returns: strengths, issues (Critical/Important/Minor), assessment

## Responding to review

1. Read all feedback completely
2. For each suggestion: verify it against the actual codebase before implementing
3. Fix Critical issues immediately
4. Fix Important issues before merge
5. Note Minor issues for later
6. Push back with technical reasoning when the reviewer is wrong

## Don't

- Performatively agree ("Great catch! You're absolutely right!")
- Blindly implement suggestions without verifying they're correct
- Add extras that weren't needed (YAGNI)
- Ignore feedback because you disagree — articulate why instead
```

**Step 2: Verify**

Run: `wc -w skills/review/SKILL.md`
Expected: Under 500 words (excluding frontmatter)

**Step 3: Commit**

```bash
git add skills/review/SKILL.md
git commit -m "feat: add review skill — request and respond to code reviews"
```

---

## Task 6: Create `teams` SKILL.md

**Files:**
- Create: `skills/teams/SKILL.md`

**Step 1: Write the skill file**

```markdown
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
```

**Step 2: Verify**

Run: `wc -w skills/teams/SKILL.md`
Expected: Under 500 words (excluding frontmatter)

**Step 3: Commit**

```bash
git add skills/teams/SKILL.md
git commit -m "feat: add teams skill — agent team orchestration patterns"
```

---

## Task 7: Create `execute` SKILL.md

**Files:**
- Create: `skills/execute/SKILL.md`

**Step 1: Write the skill file**

```markdown
---
name: execute
description: Execute implementation plans using agent teams for parallel work and sequential agents for dependent tasks.
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

## Checkpoints — every 3-5 tasks

- Run full test suite
- Dispatch code review via `review` skill
- Fix issues before proceeding to next batch
- Report: what was done, verification output, issues found

## After all tasks

- Final full verification: test suite, linter, build
- Final code review of complete implementation
- Hand off to `git-flow` skill for branch completion

## When to stop

- Blocker mid-batch → stop, report, get guidance
- Plan has gaps → surface them, don't guess
- Verification fails repeatedly → use `debug` skill, don't force it
```

**Step 2: Verify**

Run: `wc -w skills/execute/SKILL.md`
Expected: Under 500 words (excluding frontmatter)

**Step 3: Commit**

```bash
git add skills/execute/SKILL.md
git commit -m "feat: add execute skill — plan execution with teams and checkpoints"
```

---

## Task 8: Create `brainstorm` SKILL.md

**Files:**
- Create: `skills/brainstorm/SKILL.md`

**Step 1: Write the skill file**

```markdown
---
name: brainstorm
description: Turn rough ideas into design specs through deep collaborative interview. Use before writing code or plans.
---

# Brainstorm

Act as a PM and senior developer. Become deeply familiar with the codebase. Interview the user about implementation, UX, concerns, and tradeoffs — then write a complete design spec.

## Process

1. **Explore context** — read relevant files, docs, recent commits. Understand the project before asking questions.

2. **Interview deeply** — one question at a time via `AskUserQuestion`. Prefer multiple choice. Ask non-obvious questions: edge cases, failure modes, scaling concerns, UX implications, maintenance burden. Continue until the design is fully understood.

3. **Propose approaches** — present 2-3 alternatives with a recommendation and reasoning. Let the user choose.

4. **Write the design spec** — save to `docs/plans/YYYY-MM-DD-<topic>-design.md`. Include: goal, constraints, architecture, components, data flow, error handling, testing approach, open questions resolved during interview.

5. **Review** — use `md-review-plus <file> --review` for browser-based review. Fall back to `AskUserQuestion` if md-review-plus is not available. Iterate until approved.

6. **Hand off** — "Ready to plan implementation?" → `plan` skill.

## Principles

- One question at a time, never multiple
- Multiple choice preferred
- YAGNI ruthlessly — if it might not be needed, don't include it
- Write the complete spec in one shot, not section-by-section drip-feeds
```

**Step 2: Verify**

Run: `wc -w skills/brainstorm/SKILL.md`
Expected: Under 500 words (excluding frontmatter)

**Step 3: Commit**

```bash
git add skills/brainstorm/SKILL.md
git commit -m "feat: add brainstorm skill — collaborative design interviews"
```

---

## Task 9: Create `plan` SKILL.md

**Files:**
- Create: `skills/plan/SKILL.md`

**Step 1: Write the skill file**

```markdown
---
name: plan
description: Create detailed implementation plans with bite-sized tasks. Exact file paths, complete code, verification steps.
---

# Plan

Create implementation plans assuming the engineer has zero codebase context. Every task is bite-sized (2-5 minutes), with exact file paths, complete code, and verification commands.

## Process

1. **Understand requirements** — read the design doc or gather requirements directly
2. **Explore codebase** — find relevant patterns, dependencies, existing code
3. **Break into tasks** — each task is one action (write test, run test, implement, verify, commit)
4. **Write the plan** — save to `docs/plans/YYYY-MM-DD-<feature>-plan.md`
5. **Review** — `md-review-plus <file> --review` or `AskUserQuestion` fallback. Iterate until approved.
6. **Hand off** — "Ready to execute?" → `execute` skill

## Plan header

Every plan starts with:

```
# [Feature Name] Implementation Plan

**Goal:** [One sentence]
**Architecture:** [2-3 sentences]
**Tech Stack:** [Key technologies]
```

## Task format

```
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.ext`
- Modify: `exact/path/to/existing.ext:123-145`
- Test: `tests/exact/path/to/test.ext`

**Step 1: Write failing test**
[Complete code]

**Step 2: Run test, verify failure**
Run: `[exact command]`
Expected: FAIL with "[specific message]"

**Step 3: Implement**
[Complete code]

**Step 4: Run test, verify pass**
Run: `[exact command]`
Expected: PASS

**Step 5: Commit**
`git commit -m "feat: [description]"`
```

## Rules

- Exact file paths, always
- Complete code, not pseudo-code or "add validation here"
- Exact commands with expected output
- DRY, YAGNI, TDD, frequent commits
```

**Step 2: Verify**

Run: `wc -w skills/plan/SKILL.md`
Expected: Under 500 words (excluding frontmatter)

**Step 3: Commit**

```bash
git add skills/plan/SKILL.md
git commit -m "feat: add plan skill — bite-sized implementation plans"
```

---

## Task 10: Create `git-flow` SKILL.md

**Files:**
- Create: `skills/git-flow/SKILL.md`

**Step 1: Write the skill file**

```markdown
---
name: git-flow
description: Worktree setup and branch completion lifecycle. Start isolated work, finish with merge/PR/keep/discard.
---

# Git Flow

One skill for the full branch lifecycle: create an isolated worktree, do the work, finish with your choice of merge, PR, keep, or discard.

## Starting work — worktree setup

1. Check for existing worktree directory: `.worktrees` first, then `worktrees`
2. Check CLAUDE.md for directory preference
3. Ask user if unclear
4. Verify `.gitignore` covers the worktree directory
5. Create: `git worktree add <path> -b <branch-name>`
6. Auto-detect and run setup (`npm install`, `pip install -e .`, etc.)
7. Run tests for a clean baseline

## Finishing work — branch completion

1. Verify all tests pass — stop if they don't
2. Present four options:
   - **Merge** to base branch locally
   - **Push and create PR** via `gh pr create`
   - **Keep** branch as-is for later
   - **Discard** — requires typing "discard" to confirm
3. Execute the chosen option
4. Clean up worktree for merge and discard options
```

**Step 2: Verify**

Run: `wc -w skills/git-flow/SKILL.md`
Expected: Under 500 words (excluding frontmatter)

**Step 3: Commit**

```bash
git add skills/git-flow/SKILL.md
git commit -m "feat: add git-flow skill — worktree lifecycle management"
```

---

## Task 11: Create README.md

**Files:**
- Create: `README.md`

**Step 1: Write the README**

```markdown
# Superwisdom

Streamlined development workflow skills for Claude Code. Professional, team-oriented, lean.

A consolidated replacement for [superpowers](https://github.com/obra/superpowers) — same proven workflows, fewer skills, direct tone.

## Install

```
/plugin install superwisdom@seiraiyu-marketplace
```

## Skills

| Skill | Purpose |
|-------|---------|
| **boot** | Meta-skill — check for matching skills before any task |
| **brainstorm** | Turn rough ideas into design specs through collaborative interview |
| **plan** | Create bite-sized implementation plans with exact paths, code, and verification |
| **execute** | Execute plans using agent teams (parallel) and sequential agents (dependent) |
| **tdd** | Red-green-refactor with anti-patterns and async timing guidance |
| **debug** | Four-phase: investigate → trace root cause → fix → harden |
| **review** | Request and respond to code reviews |
| **teams** | Agent team orchestration patterns |
| **git-flow** | Worktree setup and branch completion lifecycle |
| **verify** | Evidence before assertions — run proof before claiming done |

## Design Principles

1. **Blunt, not dramatic.** State what to do and why. No fear-mongering.
2. **Full documents, not drip-feeds.** Complete specs reviewed in one shot.
3. **Teams over subagents.** `TeamCreate` for parallel work, `Agent` for quick dispatches.
4. **Consolidated.** 10 skills instead of 20. One debugging skill, not three.
5. **Lean.** <150 words for boot, <200 for verify, <500 for everything else.

## Dependencies

- **md-review-plus** — used by brainstorm and plan for document review (graceful fallback to AskUserQuestion)
- **Claude Code agent teams** — used by execute and teams skills
- **gh CLI** — used by git-flow for PR creation

## License

MIT
```

**Step 2: Verify**

Run: `wc -w README.md`
Expected: Reasonable length, complete information

**Step 3: Commit**

```bash
git add README.md
git commit -m "docs: add README with skill overview and install instructions"
```

---

## Summary

| Task | Skill | Word Target | Dependencies |
|------|-------|-------------|--------------|
| 1 | boot | <150 | None |
| 2 | verify | <200 | None |
| 3 | tdd | <500 | None |
| 4 | debug | <500 | None |
| 5 | review | <500 | None |
| 6 | teams | <500 | None |
| 7 | execute | <500 | None |
| 8 | brainstorm | <500 | None |
| 9 | plan | <500 | None |
| 10 | git-flow | <500 | None |
| 11 | README | — | None |

All tasks are independent and can be parallelized. Recommended batch: all 11 tasks in one pass, since each is a single file write with no code dependencies.
