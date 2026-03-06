---
name: verify
description: Run verification before claiming work is done. Evidence before assertions.
allowed-tools: Bash Read
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

## Say

- "Tests pass — here's the output"
- "Build exits 0 — confirmed just now"
- "Bug fix verified — reproduction test passes"

## Reference

| Claim | Requires | Not sufficient |
|-------|----------|----------------|
| Tests pass | Test output: 0 failures | Previous run |
| Build succeeds | Build exit 0 | Linter passing |
| Bug fixed | Reproduction test passes | "Code looks right" |
| Linter clean | Linter output: 0 errors | Partial file check |
