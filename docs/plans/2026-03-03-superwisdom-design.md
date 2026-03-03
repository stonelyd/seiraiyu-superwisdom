# Superwisdom — Design Specification

**Date:** 2026-03-03
**Plugin:** `superwisdom` on the Seiraiyu Marketplace
**Goal:** Full replacement for `obra/superpowers` — same proven workflows, streamlined delivery, team-oriented, integrated with `md-review-plus`.

## Why Fork

Superpowers skills are well-designed but:

1. **Too verbose.** Section-by-section drip-feeding, excessive ceremony, repeated rationalization tables across multiple skills.
2. **Subagent-centric.** Uses `subagent-driven-development` and `dispatching-parallel-agents` instead of Claude Code's native agent teams.
3. **Aggressive tone.** "YOU WILL FAIL", "EXTREMELY IMPORTANT", "not negotiable" — effective for enforcement but exhausting for daily use.

Superwisdom keeps the proven workflows and consolidates 20 skills → 10 with a professional, direct tone.

## Design Principles

1. **Blunt, not dramatic.** State what to do and why. No fear-mongering. Assume the agent is a competent senior developer.
2. **Full documents, not drip-feeds.** Write complete specs/plans, review via `md-review-plus` in one shot instead of presenting 200-word sections.
3. **Teams over subagents.** Use `TeamCreate` and agent teams for parallel execution. Reserve `Agent` tool subagents for quick single-purpose dispatches (research, review).
4. **Consolidate related skills.** One debugging skill, not three. One review skill, not two. Reduce context window overhead.
5. **Lean skill files.** Target <300 words for boot/frequently-loaded skills, <500 words for others. No repeated rationalization tables — one shared pattern.

## Skill Inventory (10 Skills)

### 1. boot

**Replaces:** using-superpowers

**Purpose:** Meta-skill loaded on every conversation. Ensures skill-checking happens before any task.

**Key changes from superpowers:**
- Drop "EXTREMELY IMPORTANT", "you WILL fail", rationalization list
- Keep: scan skills → match → load → announce → follow
- Keep: TodoWrite for checklists
- Tone: "Check for matching skills before starting work. If one exists, use it."
- Target: <150 words

**Workflow:**
1. Scan available skills
2. If match → load with Skill tool, announce, follow
3. If skill has checklist → TodoWrite each item

---

### 2. brainstorm

**Replaces:** brainstorming

**Purpose:** Turn rough ideas into fully-formed design specs through deep collaborative interview.

**Key changes from superpowers:**
- Replace section-by-section design presentation with: write full doc → review via `md-review-plus --review`
- Deeper interview: "Assume you are a PM and senior developer. Become expertly familiar with the codebase. Interview in detail using AskUserQuestion about implementation, UX, concerns, tradeoffs. Questions should not be obvious."
- Keep: one question at a time, multiple choice preferred, explore 2-3 approaches
- Drop: 200-300 word section limit, incremental validation of design sections

**Workflow:**
1. Explore project context (files, docs, recent commits)
2. Interview user deeply via AskUserQuestion — one question at a time, non-obvious, covering technical implementation, UX, concerns, tradeoffs
3. Continue interviewing until design is fully understood
4. Propose 2-3 approaches with recommendation and reasoning
5. Write complete design spec to `docs/plans/YYYY-MM-DD-<topic>-design.md`
6. Review via `md-review-plus <file> --review` — user approves/rejects/comments in browser
7. Iterate based on feedback until approved
8. Ask: "Ready to plan implementation?" → hand off to `plan` skill

**Design doc includes:** Goal, constraints, architecture, components, data flow, error handling, testing approach, open questions resolved during interview.

---

### 3. plan

**Replaces:** writing-plans

**Purpose:** Create detailed implementation plans with bite-sized tasks that any engineer can execute with zero codebase context.

**Key changes from superpowers:**
- Review via `md-review-plus --review` instead of interactive presentation
- Keep: exact file paths, complete code examples, verification steps per task
- Keep: TDD structure per task (failing test → implement → verify)
- Add: phase tracking with status columns (pending/in-progress/done/tested/pushed)
- Drop: verbose plan header template, redundant execution handoff section

**Workflow:**
1. Read design doc (from brainstorm) or understand requirements
2. Explore codebase for relevant patterns, dependencies, existing code
3. Break work into bite-sized tasks (2-5 minutes each)
4. Each task includes:
   - Exact file paths
   - Complete code (not pseudo-code)
   - Failing test first
   - Verification command
   - Commit message
5. Write plan to `docs/plans/YYYY-MM-DD-<feature>-plan.md`
6. Review via `md-review-plus <file> --review`
7. Iterate until approved
8. Ask: "Ready to execute?" → hand off to `execute` skill

**Plan format includes:** Goal, architecture summary, task list with phase/status tracking, dependencies between tasks.

---

### 4. execute

**Replaces:** executing-plans + subagent-driven-development

**Purpose:** Execute implementation plans using agent teams for parallel work and subagents for sequential/review work.

**Key changes from superpowers:**
- Primary execution via `TeamCreate` + agent teams for independent tasks
- Fall back to sequential Agent subagents only for tasks with dependencies
- Code review via `superpowers:code-reviewer` agent (or equivalent) between batches, not after every single task
- Keep: batch execution with checkpoints, TodoWrite tracking, verification after each task
- Drop: "fresh subagent per task" requirement, excessive review ceremony

**Workflow:**
1. Load plan, review critically — raise concerns before starting
2. Create TodoWrite tracker for all tasks
3. Identify independent vs dependent task groups
4. For independent tasks: `TeamCreate` → assign tasks to team agents → parallel execution
5. For dependent tasks: execute sequentially
6. After each batch (3-5 tasks): checkpoint report, run full test suite, code review
7. Fix issues before proceeding to next batch
8. After all tasks: final verification, hand off to `git-flow` skill

**Team structure:**
- Team lead (you) coordinates via task list
- Worker agents execute individual tasks in worktree isolation
- Review agent validates after each batch

---

### 5. tdd

**Replaces:** test-driven-development + testing-anti-patterns + condition-based-waiting

**Purpose:** Red-green-refactor discipline with built-in anti-pattern prevention and timing guidance.

**Key changes from superpowers:**
- Merge three skills into one
- Keep: iron law (no production code without failing test), red-green-refactor cycle, anti-pattern gates
- Absorb testing-anti-patterns as a "Don't" section within TDD
- Absorb condition-based-waiting as a "Timing" section for async tests
- Trim rationalization table to top 5 most common excuses (not 15+)
- Drop: extensive persuasion psychology, repeated red flags sections

**Workflow:**
1. **RED:** Write minimal failing test. Run it. Confirm it fails for the right reason.
2. **GREEN:** Write simplest code to pass. Run test. Confirm pass.
3. **REFACTOR:** Clean up — remove duplication, improve names, extract patterns. All tests still pass.
4. Repeat for next behavior.

**Anti-patterns (integrated):**
- Don't test mock behavior — test real code through mocks
- Don't add test-only methods to production code
- Don't mock without understanding what you're replacing
- Don't use `setTimeout`/`sleep` — use condition-based waiting: `waitFor(() => condition)`

**Condition-based waiting (integrated):**
- Replace `sleep(N)` with polling: check condition every 10ms, timeout at 5s
- Wait for state, then optionally add documented delay — never reverse order

---

### 6. debug

**Replaces:** systematic-debugging + root-cause-tracing + defense-in-depth

**Purpose:** One debugging flow: investigate → trace root cause → fix → validate at boundaries.

**Key changes from superpowers:**
- Merge three skills into four phases of one workflow
- Keep: systematic 4-phase approach, backward tracing, multi-layer validation
- Drop: repeated rationalization sections, separate skill overhead
- Trim: verbose examples, keep one clear example per phase

**Workflow:**

**Phase 1 — Investigate:**
- Read error messages completely
- Reproduce consistently
- Check recent changes (`git log`, `git diff`)
- Gather evidence at component boundaries

**Phase 2 — Trace Root Cause:**
- Start at symptom, trace backward through call stack
- Ask "what called this?" at each level until you find where bad data originates
- Add instrumentation (console.error with stack traces) if manual tracing stalls
- Fix at source, not at symptom

**Phase 3 — Fix:**
- Form single hypothesis
- Make smallest possible change
- Write failing test that reproduces the bug
- Implement fix, verify test passes
- If 3+ fixes fail: stop and question architecture

**Phase 4 — Harden:**
- Validate at every layer data passes through (defense-in-depth)
- Entry point validation → business logic checks → environment guards → debug instrumentation
- Make the bug structurally impossible, not just patched

---

### 7. review

**Replaces:** requesting-code-review + receiving-code-review

**Purpose:** Code review — requesting it and responding to feedback — in one skill.

**Key changes from superpowers:**
- Merge request/receive into one skill with two modes
- Keep: dispatch reviewer agent, structured feedback format, pushback protocol
- Drop: repeated performative-agreement warnings, verbose examples
- Trim: response protocol to essential steps

**Requesting review:**
1. Get git SHAs for the diff range
2. Dispatch code-reviewer agent with: what was implemented, plan/requirements reference, diff range
3. Reviewer returns: strengths, issues (Critical/Important/Minor), assessment

**Receiving review:**
1. Read all feedback
2. Verify each suggestion against actual codebase — don't blindly implement
3. Fix Critical issues immediately, Important before merge, note Minor
4. Push back with technical reasoning if reviewer is wrong
5. Never: performative agreement ("You're absolutely right!"), blind implementation, YAGNI violations disguised as "professional" additions

---

### 8. teams

**Replaces:** dispatching-parallel-agents

**Purpose:** Patterns for orchestrating Claude Code agent teams effectively.

**Key changes from superpowers:**
- Built around `TeamCreate` / `SendMessage` / task list coordination (Claude Code native teams)
- Not about subagent dispatch — about team coordination
- When to use teams vs single agent vs subagent dispatch
- Drop: subagent-specific dispatch patterns

**When to use teams:**
- 3+ independent tasks that can run in parallel
- Multi-file changes where agents won't conflict
- Large implementation plans with clear task boundaries

**When NOT to use teams:**
- Tasks with shared state or sequential dependencies
- Quick single-purpose dispatches (use Agent subagent instead)
- Tasks touching the same files (merge conflict risk)

**Team patterns:**
- **Parallel workers:** TeamCreate → assign independent tasks → each agent works in worktree isolation → review merged results
- **Research swarm:** Multiple agents investigate different aspects, report findings to lead
- **Pipeline:** Sequential handoff between specialized agents (design → implement → test → review)

**Coordination:**
- Use task list as single source of truth
- Lead assigns tasks via TaskUpdate
- Workers mark complete and check for next task
- Broadcast only for blocking issues — DM for everything else

---

### 9. git-flow

**Replaces:** using-git-worktrees + finishing-a-development-branch

**Purpose:** Worktree setup and branch completion in one lifecycle skill.

**Key changes from superpowers:**
- Merge worktree creation and branch finishing into one skill
- Keep: directory priority, .gitignore verification, auto-detect setup, 4 completion options
- Drop: verbose directory selection explanation, redundant quick reference table

**Starting work (worktree):**
1. Check for existing worktree directory (.worktrees > worktrees)
2. Check CLAUDE.md for preference
3. Ask user if needed
4. Verify .gitignore covers worktree directory
5. Create: `git worktree add <path> -b <branch>`
6. Auto-detect and run setup (npm install, etc.)
7. Run tests for clean baseline

**Finishing work (branch completion):**
1. Verify all tests pass — stop if not
2. Present 4 options:
   - Merge to base branch locally
   - Push and create PR
   - Keep branch as-is
   - Discard (requires "discard" confirmation)
3. Execute chosen option
4. Clean up worktree (for merge and discard)

---

### 10. verify

**Replaces:** verification-before-completion

**Purpose:** Run verification before claiming anything is done. Evidence before assertions.

**Key changes from superpowers:**
- Keep: iron law (no claims without fresh verification), evidence-first principle
- Trim: rationalization table to top 5 shortcuts
- Drop: verbose failure examples, keep one clear table
- Target: <200 words — this skill should be fast to load and act on

**Rule:** Before claiming work is complete, fixed, or passing:
1. Identify what command proves the claim
2. Run it fresh (not from cache or memory)
3. Read full output, check exit code
4. Only then make the claim with evidence

**Applies to:** test results, build success, bug fixes, linter output, deployment status, agent reports.

**No:** "should work", "probably passes", "seems fixed", "tests were passing earlier". Run it. Show output.

---

## Plugin Structure

```
superwisdom/
├── plugin.json
├── LICENSE
├── README.md
└── skills/
    ├── boot/
    │   └── SKILL.md
    ├── brainstorm/
    │   └── SKILL.md
    ├── plan/
    │   └── SKILL.md
    ├── execute/
    │   └── SKILL.md
    ├── tdd/
    │   └── SKILL.md
    ├── debug/
    │   └── SKILL.md
    ├── review/
    │   └── SKILL.md
    ├── teams/
    │   └── SKILL.md
    ├── git-flow/
    │   └── SKILL.md
    └── verify/
        └── SKILL.md
```

## plugin.json

```json
{
  "name": "superwisdom",
  "version": "1.0.0",
  "description": "Streamlined development workflow skills. Professional, team-oriented, lean.",
  "skills": [
    "skills/boot",
    "skills/brainstorm",
    "skills/plan",
    "skills/execute",
    "skills/tdd",
    "skills/debug",
    "skills/review",
    "skills/teams",
    "skills/git-flow",
    "skills/verify"
  ]
}
```

## Tone Guide

All skills follow this tone:

**Do:**
- "Check for matching skills before starting work."
- "Write the test first. Run it. Confirm it fails."
- "If 3+ fixes fail, stop and question the architecture."

**Don't:**
- "YOU WILL FAIL if you don't check for skills."
- "This is NOT NEGOTIABLE. You CANNOT rationalize your way out."
- "EXTREMELY IMPORTANT: If you think there's even a 1% chance..."

Assume the agent is a senior developer who follows instructions when they're clear and well-reasoned. State what to do, why it matters, and move on.

## Dependencies

- **md-review-plus:** Used by brainstorm and plan skills for document review. Graceful fallback to AskUserQuestion-based review if not installed.
- **Claude Code agent teams:** Used by execute and teams skills.
- **gh CLI:** Used by git-flow (PR creation).

## Resolved Decisions

1. **Naming:** Short names (tdd, debug, verify, etc.) — faster to type, less context overhead.
2. **md-review-plus:** Graceful fallback. Try `md-review-plus --review` first, fall back to AskUserQuestion if not available.
3. **Skill-dev skills:** Dropped for v1. Can add later if needed.

## Open Questions

1. Should `boot` skill be a session hook instead of a skill? (Superpowers uses it as both — the skill gets injected via hook AND is loadable as a skill.)

## Migration from Superpowers

1. Install superwisdom: `/plugin install superwisdom@seiraiyu-marketplace`
2. Uninstall superpowers: `/plugin uninstall superpowers`
3. Skills map 1:1 — no workflow changes, just consolidated and streamlined
