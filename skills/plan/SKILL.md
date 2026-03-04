---
name: plan
description: Create detailed implementation plans with bite-sized tasks. Exact file paths, complete code, verification steps.
---

# Plan

Assume the engineer executing this plan has zero codebase context. Give them everything: exact file paths, complete code, verification commands, expected output. Every task is one action, 2-5 minutes.

## Process

1. Read the design doc. Explore the codebase for patterns, dependencies, existing code.
2. Break work into bite-sized tasks. Each task = one action (write test, run test, implement, verify, commit).
3. If anything is ambiguous, use `AskUserQuestion` to clarify before writing the plan.
4. Write the full plan to `docs/plans/YYYY-MM-DD-<feature>-plan.md`.
5. Review with `md-review-plus <file> --review`. Iterate until approved.
6. After plan is written and reviewed, present the handoff:

```
Plan complete: `docs/plans/YYYY-MM-DD-<feature>-plan.md`

To execute with a clean context, run `/clear` then paste:

/execute docs/plans/YYYY-MM-DD-<feature>-plan.md
```

Use the actual plan file path. The user copies one line after clearing.

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
| 2 | ... | pending | no | no |
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
