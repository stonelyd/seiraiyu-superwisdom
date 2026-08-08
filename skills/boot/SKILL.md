---
name: boot
description: Meta-skill loaded on every conversation. Check for matching skills before starting any task.
allowed-tools: Skill Read
---

# Boot

Check for matching skills before starting work. If one exists, use it.

## Before any task

1. Scan available skills
2. If a skill matches → load it with the Skill tool, announce what you're doing ("I'm using the tdd skill to implement this feature test-first"), and follow it
3. If the skill has a checklist → create a tracked task (TaskCreate) for each item. Don't work through checklists mentally — tracked items don't get skipped.

If no skill matches, proceed normally.
