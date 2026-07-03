# Goal-Oriented Brainstorm → Plan Variant — Design

**Date:** 2026-07-02
**Status:** DRAFT (pending md-review-plus)
**Repos touched:** `seiraiyu-superwisdom` (new skills), `seiraiyu-marketplace` (registry), `seiraiyu-skills` (submodule pointers)
**Author:** David Stonely

---

## 1. Goal (the `/goal` statement)

This design dogfoods the very pattern it introduces. Its own completion condition, written proof-command-embedded so `/goal`'s evaluator can judge it from surfaced output:

> **`/goal` The goal-oriented variant is shipped: `skills/brainstorm-goal/SKILL.md` and `skills/plan-goal/SKILL.md` both exist in the seiraiyu-superwisdom repo AND `python -c "import yaml,glob,sys; [yaml.safe_load(open(f).read().split('---')[1]) for f in ['skills/brainstorm-goal/SKILL.md','skills/plan-goal/SKILL.md']]"` prints no error (both have valid frontmatter with name/description/allowed-tools) AND `grep -l 'plan-goal' skills/brainstorm-goal/SKILL.md` matches (brainstorm-goal hands off to plan-goal, not plan) AND `grep -c '/goal' skills/plan-goal/SKILL.md` is ≥3 (plan-goal's handoff emits a `/goal` line) AND `plugin.json` version is `1.2.0` AND `marketplace.json`'s superwisdom entry version is `1.2.0` and its description names the goal variants AND `git status` in each of the three repos shows the intended files staged/committed and nothing unexpected modified. No existing skill (brainstorm, plan, execute, …) is altered.**

### Machine-verifiable acceptance criteria (full checklist)

| # | Criterion | Proof command | Pass condition |
|---|-----------|---------------|----------------|
| 1 | brainstorm-goal skill exists with valid frontmatter | `head -6 skills/brainstorm-goal/SKILL.md` | shows `name: brainstorm-goal`, `description:`, `allowed-tools:` |
| 2 | plan-goal skill exists with valid frontmatter | `head -6 skills/plan-goal/SKILL.md` | shows `name: plan-goal`, `description:`, `allowed-tools:` |
| 3 | brainstorm-goal terminates into plan-goal | `grep -c 'plan-goal' skills/brainstorm-goal/SKILL.md` | ≥1 |
| 4 | plan-goal emits a dual copy-paste handoff | `grep -c '/goal' skills/plan-goal/SKILL.md` | ≥3 |
| 5 | plan-goal handoff includes `/execute` and `/clear` | `grep -E '/(execute|clear)' skills/plan-goal/SKILL.md` | both present |
| 6 | Originals untouched | `git -C seiraiyu-superwisdom status --porcelain skills/brainstorm skills/plan skills/execute` | empty |
| 7 | plugin.json bumped | `grep '"version"' skills/../.claude-plugin/plugin.json` | `1.2.0` |
| 8 | marketplace.json superwisdom entry bumped + described | `python3 -c "import json;d=json.load(open('.claude-plugin/marketplace.json'));p=[x for x in d['plugins'] if x['name']=='superwisdom'][0];print(p['version']); print('goal' in p['description'])"` | prints `1.2.0` then `True` |
| 9 | README lists new skills | `grep -E 'brainstorm-goal\|plan-goal' seiraiyu-superwisdom/README.md` | both matched |
| 10 | Skills load without error | `/doctor` or plugin reload shows superwisdom with brainstorm-goal + plan-goal, no parse errors | listed, no errors |

The inline `/goal` string in §1 is the distilled, proof-command-embedded version handed to the user. This table is the human/agent reference. This is the "two-tier in artifacts, proof-command style inline" resolution.

---

## 2. Problem

Today the brainstorm → plan → execute trilogy ends the plan step with a single handoff line:

```
/execute docs/plans/YYYY-MM-DD-<feature>-plan.md
```

David already requires (memory `design-docs-need-goal-section`) that **every design doc carry a verifiable Goal section** so `/goal` can run alongside `/execute`. But the plan skill's handoff **never surfaces a copy-pasteable `/goal` line**. The user has to hand-author the `/goal` condition from the design doc every time. The friction: *there is no single, convenient, copy-and-paste `/execute <file>` + `/goal <statement>` block.*

## 3. What `/goal` actually is (grounding)

`/goal` is a **built-in Claude Code slash command** (requires v2.1.139+). Confirmed behavior ([docs](https://code.claude.com/docs/en/goal)):

- Takes an **inline condition string** (≤4000 chars). **Not a file path.**
- After every turn a small fast model checks whether the condition holds; if not, Claude takes another turn autonomously. Auto-clears when met. `/goal` (no arg) shows progress; `/goal clear` cancels. One goal per session.
- **Critical:** the evaluator judges the condition **against what Claude has surfaced in the conversation — it does not run commands or read files itself.** So the condition must be phrased as something Claude's own output can demonstrate.
- A good condition = **one measurable end state + a stated check (the proof command) + constraints that must not change.**

**Consequence for design:** a giant numbered checklist pasted into `/goal` is counterproductive — the evaluator can't execute it. The right artifact is a **distilled inline condition that names the exact commands `/execute` must run and surface, their expected results, and the scope constraints.** The detailed checklist stays in the design doc `§Goal` for humans and for Claude to work against.

**Also confirmed:** Claude cannot run `/goal` itself — it is always user-invoked. Therefore there is **no `execute-goal` skill**; the existing `execute` is reused unchanged, and the user pastes `/goal` manually.

## 4. Decisions (from interview)

| ID | Decision |
|----|----------|
| D1 | **Two new skills**, `brainstorm-goal` + `plan-goal`, as an opt-in parallel family (mirrors how `superwisdom-db` is a parallel family). Originals untouched. |
| D2 | **No `execute-goal`.** Claude can't self-run `/goal`; `execute` is reused as-is. |
| D3 | `/goal` argument is **inline text**, not a file path (built-in command constraint). |
| D4 | Inline `/goal` condition is authored **proof-command-embedded**: names exact commands `/execute` must run + surface, expected results, and scope constraints. |
| D5 | **Two-tier artifacts:** design doc `§1 Goal` holds the full machine-verifiable checklist; `plan-goal` distills it into the inline `/goal` string. |
| D6 | The change **spans both repos**: `seiraiyu-superwisdom` (skills) and `seiraiyu-marketplace` (registry), plus `seiraiyu-skills` submodule-pointer bumps. |
| D7 | Version realignment: `plugin.json` (1.1.0) and `marketplace.json` superwisdom entry (1.0.0) are **inconsistent today**; both set to **1.2.0**. |

## 5. Architecture

```
brainstorm-goal  (new)                 plan-goal  (new)                 execute (existing, unchanged)
─────────────────                      ────────────                      ──────────────────────────
explore → interview → propose          read design §1 Goal               user runs after /clear:
  → write design doc                   → break into tasks                  /goal <distilled condition>
    §1 = machine-verifiable Goal       → write plan                        /execute <plan.md>
    (proof-command checklist)          → md-review-plus
  → md-review-plus                     → DISTILL §1 Goal into an
  → commit                               inline proof-command /goal
  → hand off to  ──────────────────►     condition
    plan-goal                          → emit DUAL copy-paste handoff
```

### 5.1 `brainstorm-goal`

A near-clone of `brainstorm` with three deltas:

1. **Mandate §1 Goal.** The design spec's first section MUST be a machine-verifiable Goal: a proof-command checklist (table of criterion / proof command / pass condition) plus a one-paragraph distilled inline condition. Not prose. This satisfies memory `design-docs-need-goal-section`.
2. **Terminal handoff is `plan-goal`**, not `plan`.
3. Everything else identical: deep interview, `md-review-plus` mandatory, commit the design doc, YAGNI. Reuses the same flow/checklist.

### 5.2 `plan-goal`

A near-clone of `plan` with two deltas:

1. **Distill the Goal.** After writing + reviewing the plan, read the design doc `§1 Goal` and distill it into ONE inline `/goal` condition (≤4000 chars), proof-command-embedded: `<end state> AND <proof command> output shows <expected> AND ... AND no files outside <scope> modified`.
2. **Dual handoff.** The final block is:

   ```
   Plan complete: docs/plans/YYYY-MM-DD-<feature>-plan.md
   Goal distilled from: docs/plans/YYYY-MM-DD-<feature>-design.md §1

   To execute goal-tracked with a clean context, run /clear then paste:

   /goal <distilled proof-command-embedded condition>
   /execute docs/plans/YYYY-MM-DD-<feature>-plan.md
   ```

   `/goal` first (sets the session condition), then `/execute` (the task Claude works on until the condition holds).

### 5.3 Why not modify the originals

Considered and rejected (D1). The originals stay lean and predictable for callers who don't want autonomous goal-looping. A parallel family is the established house pattern (`superwisdom-db`), is discoverable by name, and is trivially reversible. The memory's "every design doc needs a Goal" is still honored — `brainstorm-goal` is how you get it on demand.

## 6. Components / files

**Repo A — `seiraiyu-superwisdom`:**
- Create `skills/brainstorm-goal/SKILL.md`
- Create `skills/plan-goal/SKILL.md`
- Edit `.claude-plugin/plugin.json`: version `1.1.0` → `1.2.0`
- Edit `README.md`: add brainstorm-goal + plan-goal to the skill list

**Repo B — `seiraiyu-marketplace`:**
- Edit `.claude-plugin/marketplace.json`: superwisdom entry version `1.0.0` → `1.2.0`; description appends the goal variants (e.g. "… plus goal-oriented brainstorm-goal/plan-goal that emit a copy-paste `/goal` + `/execute` handoff").

**Repo C — `seiraiyu-skills` (parent):**
- Bump submodule pointers for `seiraiyu-superwisdom` and `seiraiyu-marketplace` after their commits; commit the pointer bump.

**Not edited:** the installed cache under `~/.claude/plugins/cache/...` (regenerated on plugin reload; per house rule edit submodules, not the cache).

## 7. Data flow

Design doc `§1 Goal` (detailed checklist) → `plan-goal` reads it → distills → inline `/goal` string in the handoff → user pastes `/goal` + `/execute` after `/clear` → `execute` runs, surfacing proof-command output → `/goal` evaluator reads that output each turn → auto-clears when the end state + proofs are satisfied.

## 8. Error handling / edge cases

- **Distilled condition too long (>4000 chars):** `plan-goal` must compress — reference the design `§Goal` by pointing Claude to re-run the named proof commands, keeping only the top 3–5 gating checks inline. Log if it drops any.
- **Design doc has no §1 Goal:** `plan-goal` refuses to emit a goal line and tells the user to run `brainstorm-goal` (or add a Goal section) first. Fail loud, don't fabricate criteria.
- **Evaluator can't verify a criterion from output:** rephrase toward proof commands Claude runs and prints. Never include criteria that require reading a file the evaluator can't see.
- **Version drift:** the Goal criteria 7/8 gate that plugin.json and marketplace.json agree at 1.2.0.

## 9. Testing approach

Primarily static/structural (skills are markdown prompts):
- Frontmatter parses (criteria 1–2).
- Handoff/termination greps (criteria 3–5).
- Originals untouched (criterion 6).
- Version + registry consistency (criteria 7–8).
- README (criterion 9).
- Manual smoke: reload plugins, `/doctor` lists both new skills without parse errors (criterion 10); optionally run `brainstorm-goal` on a trivial toy feature end-to-end and confirm the dual handoff renders.

## 10. Phase tracking

| Phase | Description | Status | Tested | Pushed |
|-------|-------------|--------|--------|--------|
| 1 | Author `skills/brainstorm-goal/SKILL.md` (mandate §1 Goal, hand off to plan-goal) | pending | no | no |
| 2 | Author `skills/plan-goal/SKILL.md` (distill Goal, dual handoff) | pending | no | no |
| 3 | Bump `plugin.json` → 1.2.0; update superwisdom README skill list | pending | no | no |
| 4 | Update `marketplace.json` (superwisdom → 1.2.0 + description) | pending | no | no |
| 5 | Structural verification (criteria 1–10) | pending | no | no |
| 6 | Commit each submodule; bump parent submodule pointers; commit parent | pending | no | no |

## 11. Out of scope (YAGNI)

- No `execute-goal` (D2).
- No changes to `debug`, `tdd`, `review`, `teams`, `git-flow`, `verify`, `boot`.
- No `.goal` file format or file-path `/goal` support (the built-in doesn't accept paths).
- No automation to auto-run `/goal` (it is user-invoked by design).
- No `superwisdom-db` goal variant in this pass (could follow later, not now).
