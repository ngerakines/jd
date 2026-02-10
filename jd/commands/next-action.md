---
description: Show prioritized next actions across all JD systems — tasks, inbox, and review items
argument-hint: "[system-code] [filter]"
allowed-tools: [Read, Glob, Grep, Bash]
---

# Next Action Dashboard

Generate a prioritized report of everything requiring attention across
Johnny.Decimal systems — combining active tasks, unprocessed inbox items, and
items needing review into a single view ordered from most urgent to least urgent.

## Arguments

`$ARGUMENTS` may contain:

- A system code (e.g., `P10`) to limit the report to a specific system.
- A context filter (`@phone`, `@laptop`, `@errands`).
- A project filter (`+GarageSale`, `+N42.12.05`).
- A timeframe filter (`this-week`).
- No arguments to show all systems with all actionable items.

## Workflow

1. **Discover** the JD root and available systems.
2. **Scan** data sources per system:
   - `00.02 Tasks/todo.txt` — active tasks
   - `00.01 Inbox/` — unprocessed item counts
   - `00.04 Needs review/` — items requiring decisions
   - `00.02 Tasks/someday.txt` — deferred item count
3. **Parse** tasks using jdtodo.txt format and filter for actionable items.
4. **Categorize** into priority buckets: overdue, due today, inbox, needs
   review, high priority, due this week, other actionable, coming up.
5. **Generate** a markdown report with one section per non-empty bucket.
6. **Summarize** with counts, blocked/waiting stats, and recommended focus.

## Priority Order (Highest to Lowest)

1. Overdue tasks
2. Due today
3. Inbox items (unknown urgency)
4. Needs review items
5. High priority `(A)` tasks
6. Due this week
7. Other actionable tasks
8. Coming up (threshold dates approaching within 14 days)

## Examples

```
/jd:next-action
/jd:next-action P10
/jd:next-action @phone
/jd:next-action +GarageSale
/jd:next-action this-week
```

## See Also

- `/jd:manage-tasks` — View, add, complete, and modify tasks
- `/jd:process-inbox` — Classify and file inbox items
