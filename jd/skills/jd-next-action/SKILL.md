---
name: jd-next-action
description: >
  Generate a prioritized "what do I need to do next?" report by aggregating
  tasks, inbox items, and review items across Johnny.Decimal systems. Use this
  skill when the user wants a daily overview, needs to decide what to work on
  next, or says things like "what do I need to do next," "what's on my plate,"
  "show my priorities," "what needs attention," "daily overview," "action
  dashboard," "what should I focus on," "what's urgent," "where should I start,"
  "what's most important right now," or "give me a status report." Also trigger
  when the user asks "what's coming up," "what's ahead this week," or "anything
  I'm forgetting."
---

# Johnny.Decimal Next Action Dashboard

This skill creates a unified, prioritized view of everything requiring attention
across all Johnny.Decimal systems. It combines active tasks (from `todo.txt`),
unprocessed inbox items, and items needing review into a single report ordered
from most urgent to least urgent, followed by a "coming up" section for items
approaching actionability.

This skill is **read-only** — it never modifies any files. It reads and reports.

For task parsing rules, read `../jd-task-manager/references/jdtodo-spec.md`
(the jdtodo.txt format specification).

---

## 1. Orientation: Discover the JD Environment

Before generating the report, build a mental map of the user's JD setup.

### 1.1 Locate the JD Root

The JD root folder is wherever the user's systems live. Common locations:

- `~/Library/Mobile Documents/com~apple~CloudDocs/JD/` (iCloud Drive)
- `~/Documents/JD/`
- `~/JD/`
- A project-specific folder the user designates

If you don't know the root, ask the user. If the user says "what should I do
next" without further context, check the most common locations above. If you
find exactly one, use it. If you find multiple or none, ask.

### 1.2 Identify Systems

List the top-level folders under the JD root. Each folder whose name matches
the pattern `[A-Z][0-9][0-9] *` is a JD system (e.g., `P10 Personal`,
`W20 Work`).

If the user has a single system (no SYS prefix), treat the entire JD root as
one system.

If the user specified a system code as an argument (e.g., "next actions for
P10"), limit scanning to that system only.

### 1.3 Load Each System's JDex

For every system you'll scan, read the JDex file at:

```
SYS/00-09 */00 */00.00 *JDex*
```

The JDex is needed to enrich task displays with human-readable descriptions
for `+AC.ID` references. If the JDex is missing, you can still generate the
report but cannot enrich JD codes — note this in the summary.

---

## 2. Scan Data Sources

For each system, locate and read these data sources. If a source doesn't exist,
skip it and note the absence.

### 2.1 Active Tasks — `00.02 Tasks/todo.txt`

Read and parse `todo.txt` using jdtodo.txt format rules (see the jdtodo-spec
reference). For each task, extract:

- State (incomplete, complete, cancelled)
- Priority (`(A)` through `(Z)`, or none)
- Creation date
- Description
- Contexts (`@context`)
- Projects (`+project`, `+AC.ID`)
- Key:value pairs (`due:`, `before:`, `t:`, `after:`, `id:`, `dep:`, `rec:`, `h:`)
- Notes (after ` --- `)

**Filter for actionable tasks.** A task is actionable if ALL of these are true:

- It is incomplete (not `x` or `~`)
- It is not hidden (`h:1` absent)
- Its threshold date has passed or is absent (`t:` date <= today, or no `t:`)
- Its `after:` date has passed or is absent (`after:` date <= today, or no `after:`)
- It does not have `@someday` context
- It does not have `due:someday`
- All `dep:` references point to completed tasks (or the referenced `id:` does
  not exist in the active task list)

**Track separately:**

- **Blocked tasks**: Incomplete tasks whose `dep:` points to an incomplete task.
  Count these for the summary but do not include in the main report.
- **Waiting tasks**: Tasks with `@waiting` context that are otherwise actionable.
  Include them in their normal bucket but count them separately for the summary.

### 2.2 Inbox — `00.01 Inbox/`

List the contents of the inbox folder. Do NOT read or classify individual items —
that is the inbox processor's job. Collect:

- Total item count
- File types present (`.md`, `.pdf`, `.jpg`, etc.)
- Date of the most recently added item (from filesystem metadata or date-prefix
  in filename)

If the inbox is empty, skip this data source entirely.

### 2.3 Needs Review — `00.04 Needs review/`

List items in the needs-review folder. For each item, collect:

- Filename
- Date added (from date-prefix in filename, or filesystem metadata)

If the folder doesn't exist or is empty, skip this data source.

### 2.4 Someday — `00.02 Tasks/someday.txt`

Count the number of tasks in `someday.txt` (if it exists). This is for the
summary statistics only — do not include someday tasks in the report body.

Also count tasks with `@someday` or `due:someday` in `todo.txt` that were
excluded by the actionability filter.

### 2.5 Coming Up — Future Threshold Tasks

Identify tasks that are NOT yet actionable but will become actionable within the
next 14 days:

- Tasks with `t:YYYY-MM-DD` where the date is > today AND <= today + 14 days
- Tasks with `after:YYYY-MM-DD` where the date is > today AND <= today + 14 days

These are not blocked or someday — they're simply waiting for their visibility
window to open.

---

## 3. Categorize into Priority Buckets

Place each item into exactly one bucket — the highest-priority bucket it
qualifies for. A task that qualifies for multiple buckets appears only in the
first matching one.

### 3.1 Bucket Definitions

Process in this order (first match wins):

**Bucket 1 — Overdue**
Actionable tasks where `due:` or `before:` date is before today.
Show how many days overdue each task is.

**Bucket 2 — Due Today**
Actionable tasks where `due:` or `before:` date equals today.

**Bucket 3 — Inbox**
Unprocessed items in `00.01 Inbox/` folders.
Shown as a count per system, not individual items.

**Bucket 4 — Needs Review**
Items in `00.04 Needs review/` folders.
Shown as individual items with filenames.

**Bucket 5 — High Priority**
Actionable tasks with `(A)` priority that did not land in Overdue or Due Today.

**Bucket 6 — Due This Week**
Actionable tasks where `due:` or `before:` is within the next 7 days (after
today, up to and including today + 7 days). Excludes tasks already placed in
a higher bucket.

**Bucket 7 — Other Actionable**
All remaining actionable tasks — `(B)` through `(Z)` priority and unprioritized
tasks that did not land in any bucket above.

**Bucket 8 — Coming Up**
Tasks with `t:` or `after:` dates within the next 14 days that are not yet
actionable. These are shown to help the user prepare.

### 3.2 Sorting Within Buckets

Within each bucket, sort tasks by:

1. Priority — `(A)` first, then `(B)`, then `(C)`, ..., then unprioritized last
2. Due date — earliest first (tasks with no due date sort last)
3. Creation date — oldest first

### 3.3 Empty Buckets

If a bucket has zero items, omit that section entirely from the report. Only
show sections that have content.

---

## 4. Generate the Report

### 4.1 Report Header

```
## Next Actions

**Date:** YYYY-MM-DD
**Systems:** [comma-separated system names, e.g., P10 Personal, W20 Work]
```

If scanning a single system, omit the "Systems" line and use the system name
in the header:

```
## Next Actions — P10 Personal

**Date:** YYYY-MM-DD
```

### 4.2 Overdue Section

```
### Overdue (N)

| Pri | Task | Due | Overdue |
|-----|------|-----|---------|
| (A) | [P10] Submit quarterly tax estimate +N42.13.02 @laptop | 2026-02-05 | 5 days |
| (B) | [P10] Review tuition bill +V06.31.02 @laptop | 2026-02-08 | 2 days |
```

Show the `[SYS]` prefix only when displaying multiple systems.

### 4.3 Due Today Section

```
### Due Today (N)

| Pri | Task |
|-----|------|
| (A) | [W20] Respond to P0 incident follow-up +G24.21.05 @work |
```

### 4.4 Inbox Section

```
### Inbox (N items)

| System | Items | Newest |
|--------|-------|--------|
| P10 Personal | 8 items (.md, .pdf, .jpg) | 2026-02-09 |
| W20 Work | 3 items (.md, .txt) | 2026-02-08 |

Run `/jd:process-inbox` to classify and file these items.
```

For single-system reports, simplify:

```
### Inbox (8 items)

8 unprocessed items (.md, .pdf, .jpg). Newest: 2026-02-09.

Run `/jd:process-inbox P10` to classify and file these items.
```

### 4.5 Needs Review Section

```
### Needs Review (N items)

| System | Item | Date |
|--------|------|------|
| P10 | 2026-01-28 Meeting notes.md | Jan 28 |
| W20 | 2026-02-03 Budget spreadsheet.xlsx | Feb 3 |
```

### 4.6 High Priority Section

```
### High Priority (N)

| Task | Due |
|------|-----|
| [P10] Clean out garage +GarageSale +N42.12.05 | 2026-05-01 |
| [W20] Review contract terms +G24.31.05 @laptop | 2026-02-15 |
```

### 4.7 Due This Week Section

```
### Due This Week (N)

| Pri | Task | Due |
|-----|------|-----|
| (B) | [P10] Schedule dentist for Hannah @phone | Feb 12 |
| (B) | [P10] Water all houseplants @home | Feb 11 |
```

### 4.8 Other Actionable Section

```
### Other Actionable (N)

| Pri | Task | Due |
|-----|------|-----|
| (B) | [P10] Sort items for garage sale +GarageSale | Feb 20 |
| (C) | [W20] Update project documentation @laptop | — |
| — | [P10] Update emergency contacts | — |
```

Show `—` for tasks with no due date.

### 4.9 Coming Up Section

```
### Coming Up (next 14 days)

| Task | Actionable | Due |
|------|------------|-----|
| [P10] Renew car registration +N42.12.01 | Jun 1 (in 5 days) | Jun 30 |
| [W20] Start Q2 planning +G24.21.02 | Feb 20 (in 10 days) | Mar 1 |

These tasks are not yet actionable but will become visible soon.
```

Show both the date the task becomes actionable (`t:` or `after:` date) and its
due date (if any). Include the "in N days" relative label.

### 4.10 JD Code Enrichment

When displaying tasks with `+AC.ID` or `+SYS.AC.ID` project tags, look up the
code in the relevant system's JDex. If found, you may append the description
after the task or use it in the table.

For example, if `N42.13.02` maps to "Tax returns & estimates" in the JDex:

```
| (A) | Submit quarterly tax estimate +N42.13.02 @laptop | 2026-04-15 |
```

The JD code itself is usually enough context. Only add the description if the
task text alone doesn't make the purpose clear.

### 4.11 Multi-System Display

When showing tasks from multiple systems:
- Prefix each task with `[SYS]` (e.g., `[P10]`, `[W20]`)
- Group inbox and needs-review items by system in their tables
- Include all systems in the summary

When showing a single system:
- Omit the `[SYS]` prefix
- Omit per-system grouping in inbox/review tables

### 4.12 Summary

Always end the report with a summary section:

```
### Summary

| | Count |
|--|-------|
| Overdue | 2 |
| Due today | 1 |
| Inbox items | 11 |
| Needs review | 2 |
| High priority | 2 |
| Due this week | 3 |
| Other actionable | 8 |
| **Total actionable** | **16** |
| | |
| Blocked | 3 |
| Waiting (@waiting) | 2 |
| Someday | 5 |
| Coming up (14 days) | 2 |

**Recommended focus:** [one sentence suggesting where to start]
```

The "Recommended focus" line should follow this logic:
- If overdue tasks exist: "Start with the N overdue tasks."
- Else if due-today tasks exist: "Handle the N tasks due today."
- Else if inbox has items: "Process inbox to surface any hidden urgencies."
- Else if high-priority tasks exist: "Focus on the N high-priority items."
- Otherwise: "No urgent items. Good time for a review or inbox check."

---

## 5. Filtering

The user may request a filtered view. Apply the filter AFTER scanning all data
sources but BEFORE bucketing.

### 5.1 Filter by System

"next actions for P10" — Only scan the specified system.

### 5.2 Filter by Context

"what can I do @phone" — Only include tasks with the specified context. Inbox
and needs-review items are still shown (they may contain relevant work).

### 5.3 Filter by Project

"what's next for +GarageSale" — Only include tasks with the specified project
tag. Inbox and needs-review items are omitted (they haven't been classified).

### 5.4 Filter by Timeframe

"what's due this week" — Only include tasks in the Overdue, Due Today, and
Due This Week buckets. Omit inbox, needs review, and lower-priority buckets.

---

## 6. Interaction Principles

### 6.1 Read-Only

This skill never modifies files. It reads task files, inbox contents, and
needs-review items, then generates a report. No writes, no moves, no edits.

### 6.2 No Duplicate Work

Do not read inbox item contents or classify them — that's the inbox processor's
job. Only count items and note their presence.

Do not modify task priorities or due dates — that's the task manager's job.

### 6.3 Actionable Suggestions

After the report, suggest relevant next steps using other skills:

- If inbox has items: "Run `/jd:process-inbox` to classify inbox items."
- If overdue tasks exist: "Use `/jd:manage-tasks` to reschedule or complete overdue tasks."
- If someday count is high: "Use `/jd:manage-tasks review` to review someday items."

### 6.4 Repeat Invocations

If the user asks for the report again during the same session, regenerate it
fresh — task states may have changed.

---

## 7. Error Handling

### 7.1 Missing Task Files

If `00.02 Tasks/todo.txt` doesn't exist for a system:
- Check if `00.02 Tasks.md` exists (legacy format). If so, note it and
  suggest migration via `/jd:manage-tasks`.
- If nothing exists, note "No task files found for [system]" in the summary.

### 7.2 Missing Inbox or Review Folders

If `00.01 Inbox/` or `00.04 Needs review/` doesn't exist, simply omit that
data source. Do not warn about structural absence — not all systems use all
standard zeros.

### 7.3 Unparseable Tasks

If a task line cannot be parsed:
- Include it in the report with a `[parse error]` flag
- Do not skip it — the user should know it exists
- Continue processing remaining tasks

### 7.4 JDex Not Found

If a system's JDex is missing:
- Generate the report without JD code enrichment
- Note in the summary: "JDex not found for [system] — JD codes not enriched."

### 7.5 No Data Found

If all data sources are empty across all systems:
- Report "Nothing to do!" with a clean summary showing all zeros
- Suggest: "Good time to check if anything needs capturing."

---

## Quick Reference: Priority Buckets

| Order | Bucket | Source | Criteria |
|-------|--------|--------|----------|
| 1 | Overdue | todo.txt | `due:` or `before:` < today |
| 2 | Due Today | todo.txt | `due:` or `before:` = today |
| 3 | Inbox | 00.01 Inbox/ | Any items present |
| 4 | Needs Review | 00.04 Needs review/ | Any items present |
| 5 | High Priority | todo.txt | `(A)` tasks not in above |
| 6 | Due This Week | todo.txt | `due:` or `before:` within 7 days |
| 7 | Other Actionable | todo.txt | All remaining actionable |
| 8 | Coming Up | todo.txt | `t:` or `after:` within 14 days |

## Quick Reference: Actionability Criteria

A task is actionable if ALL true:

- Not completed (`x`) or cancelled (`~`)
- Not hidden (`h:1` absent)
- Threshold passed (`t:` <= today, or no `t:`)
- After date passed (`after:` <= today, or no `after:`)
- No `@someday` context
- No `due:someday`
- All `dep:` resolved (blocking tasks completed or absent)
