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
- Propose the robust fix, not a shortcut or workaround
- Write a failing test that reproduces the bug
- Implement the fix, verify the test passes
- If the fix doesn't work: return to Phase 1, don't stack another guess
- If you need user context to understand the expected behavior, use `AskUserQuestion`

## Phase 4 — Harden

- Validate at every layer the data passes through
- Entry point validation → business logic checks → environment guards
- Make the bug structurally impossible, not just patched

## Stop conditions

- 3+ fix attempts failed → question the architecture, not the fix
- Each fix reveals a new problem elsewhere → you're treating symptoms
- "Quick fix now, investigate later" → investigate now
