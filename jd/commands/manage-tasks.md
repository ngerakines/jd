---
description: Manage tasks in a Johnny.Decimal system — view, add, complete, cancel, and modify tasks using jdtodo.txt format
argument-hint: "[system-code] [action] [filter]"
allowed-tools: [Read, Glob, Grep, Bash, Write]
---

# Manage Tasks

View, add, complete, cancel, and modify tasks in Johnny.Decimal `00.02 Tasks/`
folders using the jdtodo.txt format. Handles todo.txt, done.txt, and
someday.txt files.

## Arguments

`$ARGUMENTS` may contain:

- A system code (e.g., `P10`) to target a specific system's tasks.
- An action keyword: `add`, `complete`, `cancel`, `list`, `modify`, `review`.
- Filter criteria: priority (`A`), context (`@phone`), project (`+GarageSale`),
  JD code (`+N42.12.05`), or date range (`overdue`, `due:this-week`).
- No arguments to list all actionable tasks across all systems.

## Workflow

1. **Discover** the JD root and available systems.
2. **Locate** task files at `00.02 Tasks/` in each system (`todo.txt`,
   `done.txt`, `someday.txt`).
3. **Parse** todo.txt files using jdtodo.txt format rules.
4. **Execute** the requested action (list, add, complete, cancel, modify,
   review).
5. **Handle recurrence**: When completing a recurring task, generate the next
   occurrence and keep the original in done.txt.
6. **Archive**: Move completed and cancelled tasks to done.txt.
7. **Resolve dependencies**: Show blocked vs. actionable tasks.
8. **Summarize** changes made.

## Actions

- **list** (default): Show actionable tasks with filtering and grouping.
- **add**: Create a new task with priority, dates, contexts, projects, and
  JD codes.
- **complete**: Mark one or more tasks as done, handle recurrence, move to
  done.txt.
- **cancel**: Mark tasks as cancelled with reason, move to done.txt.
- **modify**: Change priority, add dates, add notes, update contexts or
  projects.
- **review**: Show someday items and deferred tasks for weekly review.

## Examples

```
/jd:manage-tasks
/jd:manage-tasks P10
/jd:manage-tasks P10 add
/jd:manage-tasks list @phone
/jd:manage-tasks list overdue
/jd:manage-tasks review
```
