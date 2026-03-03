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
