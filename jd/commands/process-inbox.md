---
description: Process items in a Johnny.Decimal inbox — classify, file, extract tasks, and log actions
argument-hint: "[system-code]"
allowed-tools: [Read, Glob, Grep, Bash, Write]
---

# Process Johnny.Decimal Inbox

Process items in one or more Johnny.Decimal `00.01 Inbox` folders. Each item
is read, classified against the JDex and folder structure, and moved to the
correct ID location.

## Arguments

`$ARGUMENTS` may contain a system code (e.g., `N42`) to process a specific
system's inbox. If no argument is provided, discover all systems and their
inboxes, then ask which to process.

## Workflow

1. **Discover** the JD root and available systems.
2. **Load** the target system's JDex and folder structure.
3. **List** inbox contents and report item count, types, and date range.
4. **Classify** each item using the JDex as the primary reference.
5. **File** items to their destination ID folders with date-prefix naming.
6. **Extract** tasks from items that contain action items.
7. **Log** every action to the processing log at `00.03 Processing log.md`.
8. **Summarize** what was processed, filed, and flagged.

## Classification Confidence

- **High confidence**: File without asking.
- **Medium confidence**: Recommend a destination and ask the user to confirm.
- **Low confidence**: Present 2–3 candidate locations and ask the user to choose.

## Batch Mode

For inboxes with many items, present a table of proposed filings and let the
user approve, modify, or flag specific items before executing.

## Examples

```
/jd:process-inbox
/jd:process-inbox N42
/jd:process-inbox G24
```
