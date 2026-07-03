---
name: plan-goal
description: Goal-oriented plan — create a bite-sized implementation plan AND distill the design's §1 Goal into an inline /goal condition, then emit a copy-paste /clear + /goal + /execute handoff. Pairs with brainstorm-goal.
allowed-tools: Read Glob Grep Write AskUserQuestion Bash(md-review-plus:*) Bash(git:*) Skill
---

# Plan (Goal-Oriented)

Same bite-sized planning discipline as `plan`, with one addition: after the plan is written and reviewed, **distill the design doc's §1 Goal into an inline `/goal` condition** and emit a dual copy-paste handoff so the user can run goal-tracked execution.

Assume the engineer executing this plan has zero codebase context. Give them everything: exact file paths, complete code, verification commands, expected output. Every task is one action, 2–5 minutes.

## Process

1. Read the design doc — especially its §1 Goal. Explore the codebase for patterns, dependencies, existing code.
2. Break work into bite-sized tasks. Each task = one action (write test, run test, implement, verify, commit).
3. If anything is ambiguous, use `AskUserQuestion` to clarify before writing the plan.
4. Write the full plan to `docs/plans/YYYY-MM-DD-<feature>-plan.md`.
5. Review with `md-review-plus <file> --review`. Iterate until approved.
6. **Distill the Goal** (see below) and present the dual handoff.

## Plan header

```
# [Feature Name] Implementation Plan

**Goal:** [One sentence]
**Architecture:** [2-3 sentences]
**Tech Stack:** [Key technologies]
```

## Task tracking

Every plan includes a status table at the top, below the header:

```
| Task | Description | Status | Tested | Pushed |
|------|-------------|--------|--------|--------|
| 1 | ... | pending | no | no |
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

## Distill the Goal

Read the design doc's §1 Goal. It contains a full acceptance-criteria checklist. Distill it into ONE inline `/goal` condition string that:

- Is **≤4000 chars** (`/goal` accepts inline text only — never a file path).
- Is **proof-command-embedded**: names the exact commands `/execute` must run AND surface, their expected output, and the scope constraints. `/goal`'s evaluator only judges what Claude surfaces in the conversation — it does not run commands or read files — so every clause must be demonstrable from Claude's printed output.
- Keeps only the top gating checks inline if the full checklist would blow the char budget; point Claude to re-run the named proof commands for the rest, and note anything you dropped.

If the design doc has **no §1 Goal**, do NOT fabricate one. Stop and tell the user to run `brainstorm-goal` (or add a verifiable Goal section) first.

## Handoff

After the plan is written and reviewed, present exactly this — a dual copy-paste block:

```
Plan complete: `docs/plans/YYYY-MM-DD-<feature>-plan.md`
Goal distilled from: `docs/plans/YYYY-MM-DD-<feature>-design.md` §1

To execute goal-tracked with a clean context, run `/clear` then paste:

/goal <distilled proof-command-embedded condition>
/execute docs/plans/YYYY-MM-DD-<feature>-plan.md
```

`/goal` first (sets the session completion condition), then `/execute` (the task Claude works on until the condition holds). Use the actual file paths. The user — not Claude — runs `/goal`.

## Rules

- Exact file paths, always
- Complete code, not pseudo-code
- Exact commands with expected output
- DRY, YAGNI, TDD, frequent commits
- The `/goal` condition must be provable from surfaced output, never from unread files
