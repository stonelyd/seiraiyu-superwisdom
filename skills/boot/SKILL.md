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
