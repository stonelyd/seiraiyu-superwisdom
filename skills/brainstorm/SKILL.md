---
name: brainstorm
description: Turn rough ideas into design specs through deep collaborative interview. Use before writing code or plans.
---

# Brainstorm

Assume you are a PM and senior developer. Become expertly familiar with the codebase — read the relevant files, docs, recent commits. Understand the project deeply before asking a single question.

## Interview

Use `AskUserQuestion` to interview the user about literally anything: technical implementation, UI & UX, concerns, tradeoffs, edge cases, failure modes, scaling, maintenance burden. Make sure the questions are not obvious. Be very in-depth.

- One question at a time. Never batch.
- Multiple choice preferred — provide concrete options, not open-ended "what do you think?"
- Continue interviewing continually until the design is fully understood. Don't cut it short.

## Propose the approach

When you have enough context, propose the single most robust, correct approach. Not a shortcut. Not a quick fix. Not a workaround. The right solution to the problem. Explain why it's right and what alternatives you rejected. Present via `AskUserQuestion` for approval.

## Write the spec

Write the complete design spec to `docs/plans/YYYY-MM-DD-<topic>-design.md` in one shot. Not section-by-section. The full document.

Include: goal, constraints, architecture, components, data flow, error handling, testing approach, and all decisions from the interview.

The spec must include a **phase tracking table** so anyone can tell what's been implemented, what remains, what's tested, and what's pushed:

```
| Phase | Description | Status | Tested | Pushed |
|-------|-------------|--------|--------|--------|
| 1 | ... | pending | no | no |
| 2 | ... | pending | no | no |
```

## Review

Write the full document, then review with `md-review-plus <file> --review`. User approves, rejects, or comments in the browser. Iterate until approved.

## Hand off

"Ready to plan implementation?" → `plan` skill.

## Principles

- YAGNI ruthlessly — if it might not be needed, cut it
- Write complete specs, not drip-feeds
