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
