---
name: jd-task-manager
description: >
  Manage tasks in Johnny.Decimal systems using the jdtodo.txt format —
  parsing, creating, completing, cancelling, and modifying tasks stored in
  todo.txt files at the 00.02 Tasks location. Use this skill when the user
  wants to manage their tasks, add a todo, complete a task, check what's due,
  review their task list, or says things like "show my tasks," "what's due
  today," "add a task," "mark that done," "what should I work on," "complete
  this task," "cancel that task," "show overdue tasks," "what's on my plate,"
  "review my someday list," "show blocked tasks," or "what tasks are due
  this week." Also trigger when the user says "manage my todos," "update my
  task list," "process my tasks," or references todo.txt or jdtodo.txt.
---

# Johnny.Decimal Task Manager

This skill manages tasks in Johnny.Decimal systems using the jdtodo.txt
plain-text format. It reads, creates, completes, cancels, and modifies tasks
stored in `todo.txt` files within `00.02 Tasks/` folders. It also handles
`done.txt` (completed and cancelled tasks) and `someday.txt` (deferred items).

Before processing, read `references/jdtodo-spec.md` for the complete jdtodo.txt
format specification.

---

## 1. Orientation: Discover the JD Environment

Before touching any files, build a mental map of the user's JD setup.

### 1.1 Locate the JD Root

The JD root folder is wherever the user's systems live. Common locations:

- `~/Library/Mobile Documents/com~apple~CloudDocs/JD/` (iCloud Drive)
- `~/Documents/JD/`
- `~/JD/`
- A project-specific folder the user designates

If you don't know the root, ask the user. If the user says "show my tasks"
without further context, check the most common locations above. If you find
exactly one, confirm it. If you find multiple or none, ask.

### 1.2 Identify Systems

List the top-level folders under the JD root. Each folder whose name matches
the pattern `[A-Z][0-9][0-9] *` is a JD system (e.g., `P10 Personal`,
`W20 Work`, `C40 Citywide`).

If the user has a single system (no SYS prefix), treat the entire JD root as
one system.

### 1.3 Load Each System's JDex

For every system you'll work with, read the JDex file at:

```
SYS/00-09 */00 */00.00 *JDex*
```

The JDex is needed to validate JD codes on tasks and to display human-readable
descriptions alongside `+AC.ID` references. If the JDex is missing, you can
still manage tasks but cannot validate JD code references — flag this to the
user.

### 1.4 Locate Task Files

For each system, look for task files at:

```
SYS/00-09 */00 */00.02 Tasks*/todo.txt
SYS/00-09 */00 */00.02 Tasks*/done.txt
SYS/00-09 */00 */00.02 Tasks*/someday.txt
```

**Legacy detection:** If a file named `00.02 Tasks.md` exists (instead of a
`00.02 Tasks/` directory), this is the legacy markdown-checkbox format from the
system setup workflow. See §10 for migration guidance.

If no task files exist at all, offer to create the `00.02 Tasks/` directory
with an empty `todo.txt`.

---

## 2. Parse Tasks

Each line in `todo.txt` (and `done.txt`, `someday.txt`) is a task. Parse using
these rules from the jdtodo.txt specification.

### 2.1 Logical Lines

A task is one logical line. If a line ends with `\` (backslash as the very last
character, no trailing whitespace), the next physical line is a continuation of
the current task's note. Gather all continuation lines before parsing.

Skip blank lines and any lines that are entirely whitespace.

### 2.2 Task States

Determine the task state from the line prefix:

| Prefix | State | File |
|--------|-------|------|
| `x YYYY-MM-DD` | Complete | done.txt |
| `~ YYYY-MM-DD` | Cancelled | done.txt |
| `(A)` through `(Z)` | Incomplete (with priority) | todo.txt |
| Anything else | Incomplete (no priority) | todo.txt |

### 2.3 Priority and Dates

For **incomplete tasks**:

1. If the line starts with `(X) ` where X is A–Z, extract the priority.
2. After priority (or at line start if no priority), check for a date in
   `YYYY-MM-DD` format — this is the creation date.

For **complete/cancelled tasks**:

1. After the `x ` or `~ ` marker, the first date is the completion/cancellation
   date.
2. The second date (if present) is the creation date.

### 2.4 Contexts and Projects

Scan the task body (everything before ` --- ` if a note exists) for:

- **Contexts**: tokens matching `@non-whitespace` (e.g., `@phone`, `@laptop`,
  `@waiting`, `@someday`)
- **Projects**: tokens matching `+non-whitespace` (e.g., `+GarageSale`,
  `+N42.12.05`)

**JD code detection regex** for project tags:

```
\+([A-Z]\d{2}\.)?\d{2}\.\d{2}(\+[A-Za-z0-9]+)?
```

This matches `+AC.ID`, `+SYS.AC.ID`, `+AC.ID+SUB`, and `+SYS.AC.ID+SUB`.

### 2.5 Key:Value Extensions

Extract recognized key:value pairs from the task body:

| Key | Value format | Purpose |
|-----|-------------|---------|
| `due` | `YYYY-MM-DD` or `someday` | Hard deadline |
| `before` | `YYYY-MM-DD` or `someday` | Soft deadline |
| `t` | `YYYY-MM-DD` or `someday` | Threshold / visibility gate (user deferral) |
| `after` | `YYYY-MM-DD` or `someday` | Visibility gate (external constraint) |
| `id` | Non-whitespace, no colons | Task identifier |
| `dep` | Non-whitespace, no colons | Blocked by another task |
| `sup` | Non-whitespace, no colons | Subtask of another task |
| `rec` | `[+]N[d\|w\|m\|y\|b]` | Recurrence interval |
| `h` | `1` | Hidden from default views |
| `pri` | `A`–`Z` | Preserved priority (on completed tasks) |

Preserve any unrecognized key:value pairs when modifying tasks.

### 2.6 Notes

If the task body contains ` --- ` (space-hyphen-hyphen-hyphen-space), everything
after the first occurrence is the note. The note may span multiple physical lines
via `\` continuation. Leading whitespace on continuation lines is trimmed.

Key:value pairs in the note are freeform text — do NOT parse them as task
metadata.

---

## 3. List and Filter Tasks

### 3.1 Default View

When the user asks to see their tasks without specific filters, show all
**actionable** tasks. A task is actionable if ALL of these are true:

- It is incomplete (not `x` or `~`)
- It is not hidden (`h:1` absent)
- Its threshold date has passed or is absent (`t:` date ≤ today, or no `t:`)
- Its `after:` date has passed or is absent (`after:` date ≤ today, or no
  `after:`)
- It does not have `@someday` context
- It does not have `due:someday`
- All `dep:` references point to completed tasks (or the referenced `id:` does
  not exist in the active task list)

Tasks with unresolved dependencies are shown in a separate **Blocked** section
rather than hidden entirely.

### 3.2 Filter by Context

When the user specifies a context (e.g., "show me @phone tasks"), show only
tasks that have that context tag. Common filters:

- `@phone` — calls to make
- `@laptop` — computer work
- `@errands` — out-of-house tasks
- `@waiting` — blocked on someone else
- `@someday` — not currently actionable
- `@agenda` — discuss at next meeting

### 3.3 Filter by Project or JD Code

When the user specifies a project or JD code:

- `+GarageSale` — all tasks in that named project
- `+N42.12.05` — all tasks referencing that JD ID
- `+N42.12` — all tasks referencing any ID in that JD category

### 3.4 Filter by Priority

Show tasks at or above a given priority:

- "show priority A tasks" → only `(A)` tasks
- "show high priority" → `(A)` and `(B)` tasks
- "show all priorities" → all tasks grouped by priority

### 3.5 Filter by Date

- **overdue**: Tasks where `due:` or `before:` date is before today
- **due today**: Tasks where `due:` is today
- **due this week**: Tasks where `due:` is within the next 7 days
- **upcoming**: Tasks where `due:` is within the next 14 or 30 days
- **no due date**: Tasks with no `due:` or `before:` tag

### 3.6 Dependency Resolution

Build the dependency graph from `id:`, `dep:`, and `sup:` tags:

1. Index all tasks by their `id:` value.
2. For each task with `dep:X`, check if `id:X` exists and is incomplete.
3. If `id:X` is incomplete, the task is **blocked**.
4. If `id:X` is complete (in done.txt) or doesn't exist, the dependency is
   resolved.

Show blocked tasks in a separate section with the blocking task identified:

```
### Blocked

(B) Sort items into keep/sell/donate +GarageSale dep:garage-clean
    ↳ blocked by: (A) Clean out garage for sale [garage-clean]
```

For `sup:` relationships, show subtask grouping when listing by project:

```
### +GarageSale

(A) Clean out garage for sale id:garage-clean due:2026-05-01
  ├─ (B) Sort items into keep/sell/donate sup:garage-clean
  ├─ (B) Sweep and mop garage floor sup:garage-clean
  └─ (C) Arrange sale items on tables sup:garage-clean dep:sort-items
```

### 3.7 Grouping

Default grouping is by priority. The user may request grouping by:

- **System**: Group tasks by JD system (for cross-system views)
- **Project**: Group by `+project` or `+AC.ID` tags
- **Context**: Group by `@context` tags
- **Due date**: Group into overdue / today / this week / later / no date
- **Priority**: Group by `(A)`, `(B)`, `(C)`, etc.

### 3.8 Cross-System View

When no system is specified and multiple systems have task files, merge all
tasks and show system context. Prefix each task's display with its system
code if it's identifiable from `+SYS.AC.ID` tags or from which system's
`todo.txt` it came from:

```
## All Systems — 15 active tasks

### Priority A
[P10] (A) Submit quarterly tax estimate +N42.13.02 @laptop due:2026-04-15
[W20] (A) Respond to P0 incident follow-up +G24.21.05 @work due:2026-02-10

### Priority B
[P10] (B) Schedule dentist for Hannah +H11.25.01 @phone before:2026-03-01
```

### 3.9 JD Code Enrichment

When displaying tasks with JD code project tags, look up the code in the
system's JDex and show the ID description:

```
(A) Submit quarterly tax estimate +N42.13.02 @laptop due:2026-04-15
    ↳ N42.13.02 = Tax returns & estimates
```

This helps the user understand the JD cross-reference without leaving the task
view.

---

## 4. Add Tasks

### 4.1 Constructing a Task Line

Build a well-formed jdtodo.txt line from user input. The minimum requirement is
a task description — everything else is optional. The line format is:

```
[(priority)] [creation-date] description [@contexts] [+projects] [key:values] [ --- note]
```

### 4.2 Creation Date

Always add today's date as the creation date on new tasks.

### 4.3 Priority

If the user specifies a priority, add it. If they don't mention priority, do not
add one — unprioritized tasks are valid. Only ask about priority if the task
sounds urgent or time-sensitive and the user didn't specify.

### 4.4 JD Code Integration

When the user mentions a JD location or the task clearly relates to a JD ID:

1. Add the appropriate `+AC.ID` or `+SYS.AC.ID` project tag.
2. Validate the code against the JDex. If the code doesn't exist, warn the
   user and ask whether to use it anyway or choose a different ID.
3. If the user describes a domain but doesn't specify a JD code (e.g., "add a
   task about insurance"), suggest relevant IDs from the JDex.

### 4.5 Contexts

If the task description implies a context, suggest it:

- Phone calls → `@phone`
- Computer work → `@laptop`
- Errands or physical locations → `@errands`
- Waiting on someone → `@waiting`
- Meeting discussion items → `@agenda`

Add suggested contexts only if the user confirms or if the mapping is obvious
(e.g., "call the dentist" clearly gets `@phone`).

### 4.6 Dates

When the user mentions a deadline or timing:

- "by Friday" / "due next week" → convert to `due:YYYY-MM-DD`
- "before the party" → ask for a specific date, then use `before:YYYY-MM-DD`
- "not until June" / "after surgery" → convert to `after:YYYY-MM-DD` or
  `t:YYYY-MM-DD`
- "someday" / "eventually" → use `due:someday` or `@someday`

Use `due:` for hard deadlines, `before:` for soft targets, `t:` for user
deferrals, and `after:` for external constraints. When in doubt, ask which
semantic the user intends.

### 4.7 Relationships

If the task is part of a project that has existing tasks with `id:` tags:

1. Show the existing task graph for that project.
2. Offer to set `sup:` (subtask) or `dep:` (blocked by) links.
3. If the new task should be identifiable by other tasks, suggest an `id:` —
   keep it short and descriptive (e.g., `id:post-signs`).

### 4.8 Notes

If the user provides additional context beyond the task description, add it as
a note after ` --- `. For multi-line notes, use `\` continuation:

```
(B) 2026-02-09 Schedule vet appointment @phone --- Last checkup was October. \
    Needs rabies booster.
```

### 4.9 Recurrence

If the task repeats:

- "every 3 days" → `rec:3d`
- "weekly" → `rec:1w`
- "monthly on the 1st" → `rec:+1m` (strict, anchored to date)
- "every 2 weeks" → `rec:2w`
- "every business day" → `rec:1b`

Use strict mode (`+` prefix) when the task is calendar-anchored (rent, reports).
Use non-strict when the interval from last completion matters (watering plants).

### 4.10 Which File

- New tasks go in `todo.txt` by default.
- Tasks with `@someday` or `due:someday` may optionally go in `someday.txt` if
  the user has a separate someday file. Ask the user their preference if both
  options are available.

### 4.11 Confirm Before Writing

Before adding the task, show the user the complete formatted line:

```
Adding to P10 todo.txt:
(B) 2026-02-09 Schedule vet appointment for Max +N42.25.01 @phone before:2026-03-01 --- Last checkup was October. Needs rabies booster.
```

Proceed after confirmation, or adjust based on feedback.

---

## 5. Complete and Cancel Tasks

### 5.1 Identifying the Task

When the user says "complete the insurance task" or "mark that done," identify
the target task:

1. Search active tasks for keyword matches in the description.
2. If the user references a JD code, `id:`, project, or context, use those
   to narrow the search.
3. If exactly one task matches, confirm it with the user.
4. If multiple tasks match, present the candidates and ask which one(s).
5. If no tasks match, say so and offer to list tasks for the user to choose.

### 5.2 Completing a Task

To mark a task complete:

1. Strip the `(X) ` priority from the line start (if present).
2. Prepend `x YYYY-MM-DD ` using today's date as the completion date.
3. If the task had a priority, add `pri:X` to the end of the task body
   (before any ` --- ` note).
4. Preserve all other content: description, contexts, projects, key:value
   pairs, and notes.

**Before completion:**
```
(A) 2026-02-01 Call Goodwill +GarageSale @phone due:2026-02-15
```

**After completion:**
```
x 2026-02-09 2026-02-01 Call Goodwill +GarageSale @phone due:2026-02-15 pri:A
```

### 5.3 Cancelling a Task

To cancel a task:

1. Strip the `(X) ` priority (if present).
2. Prepend `~ YYYY-MM-DD ` using today's date as the cancellation date.
3. If the task had a priority, add `pri:X`.
4. Ask the user for a cancellation reason and append it as a note.

**Before cancellation:**
```
(B) 2026-02-01 Order extra tables +GarageSale
```

**After cancellation:**
```
~ 2026-02-09 2026-02-01 Order extra tables +GarageSale pri:B --- Lauren has enough tables.
```

### 5.4 Handle Recurrence

When completing a task that has a `rec:` tag:

1. Complete the original task as described in §5.2.
2. Create a new incomplete task with the same description, projects, contexts,
   key:value pairs (except `pri:` — restore the original priority), and notes.
3. Shift date fields forward by the recurrence interval:

**Non-strict** (no `+` prefix, e.g., `rec:3d`): Calculate from today's date
(the completion date).

**Strict** (`+` prefix, e.g., `rec:+1m`): Calculate from the original `due:`
date.

4. Shift all date fields: `due:`, `t:`, `before:`, `after:`. Preserve the gap
   between `t:` and `due:` (if `t:` was 15 days before `due:`, the new `t:`
   should be 15 days before the new `due:`).
5. Set the creation date on the new task to today (the completion date).

**Example — non-strict:**
```
# Original (completed today, Feb 12):
x 2026-02-12 2026-02-09 Water all houseplants @home rec:3d due:2026-02-09 → done.txt

# New task:
(B) 2026-02-12 Water all houseplants @home rec:3d due:2026-02-15 → todo.txt
```

**Example — strict:**
```
# Original (completed today, Feb 1):
x 2026-02-01 2026-01-01 Pay rent rec:+1m due:2026-02-01 → done.txt

# New task (based on original due date, not completion date):
2026-02-01 Pay rent rec:+1m due:2026-03-01 → todo.txt
```

### 5.5 Move to done.txt

After completing or cancelling a task:

1. Remove the task line (and any `\` continuation lines) from `todo.txt`.
2. Append the completed/cancelled line to `done.txt`.
3. If `done.txt` does not exist, create it.
4. If the task was recurring, the new occurrence stays in `todo.txt`.

### 5.6 Batch Completion

When completing multiple tasks at once, present a confirmation table:

```
Complete these tasks?

| # | Task | Status |
|---|------|--------|
| 1 | Call Goodwill +GarageSale @phone | → done.txt |
| 2 | Water houseplants @home rec:3d | → done.txt + new occurrence |
| 3 | File insurance claim +N42.11.03 | → done.txt |

Proceed with all 3? [y/n/select specific]
```

---

## 6. Modify Tasks

### 6.1 Change Priority

Update or add `(X) ` at the start of the task line. If the task already has a
priority, replace it. If it has no priority, insert before the creation date
(or before the description if no creation date).

### 6.2 Add or Change Dates

Add or modify `due:`, `before:`, `t:`, `after:` tags. If the tag already
exists, replace its value. If new, append to the task body (before ` --- `).

### 6.3 Add Contexts or Projects

Append `@context` or `+project` tags to the task body. Check for duplicates
before adding.

### 6.4 Add or Update Notes

- If the task has no note, add ` --- ` followed by the note text.
- If the task already has a note, append to it (add a `\` continuation if
  needed for multiline).

### 6.5 Set Up Relationships

Add `id:`, `dep:`, or `sup:` tags to the task body:

- When adding `id:`, verify the identifier is unique within the file.
- When adding `dep:`, verify the referenced `id:` exists.
- When adding `sup:`, verify the referenced `id:` exists.

### 6.6 Move to Someday

To defer a task indefinitely:

1. Add `@someday` or change `due:` to `due:someday`.
2. If the user has a separate `someday.txt`, offer to move the task there.
3. Remove `t:` and `after:` dates if present (they're superseded by someday
   status).

### 6.7 Reactivate from Someday

To bring a task back from someday:

1. Remove `@someday` context and `due:someday` (if present).
2. If the task is in `someday.txt`, move it to `todo.txt`.
3. Ask the user for a new due date or leave undated.

---

## 7. Review Mode

Review mode is a guided walkthrough for periodic task maintenance — typically
weekly.

### 7.1 Someday Review

Present each task in `someday.txt` (or tasks with `@someday`/`due:someday` in
`todo.txt`):

For each task, offer:

- **Keep**: Leave as someday.
- **Activate**: Move to `todo.txt`, set a due date, remove `@someday`.
- **Cancel**: Mark cancelled with reason, move to `done.txt`.

### 7.2 Deferred Tasks

Show tasks whose `t:` or `after:` date has passed — these are now actionable
but may have been overlooked:

```
These tasks are now actionable:

(B) Renew car registration +N42.12.01 t:2026-06-01 due:2026-06-30
    ↳ Threshold date passed (was deferred until Jun 1)
```

Also show tasks with `t:` or `after:` dates in the next 7 days — coming up
soon.

### 7.3 Stale Tasks

Identify incomplete tasks that may be stale:

- Created more than 30 days ago with no due date and no recent modification
- No priority set
- Not part of an active project

For each stale task, offer: keep as-is, set a due date, move to someday, or
cancel.

### 7.4 Dependency Check

Show tasks whose blocking dependencies have been resolved since the last
review — these are newly actionable:

```
Newly unblocked:

(B) Post signs around neighborhood +GarageSale dep:plan-sale
    ↳ plan-sale was completed on 2026-02-05
```

### 7.5 Overdue Summary

Present all tasks past their `due:` date, sorted by how overdue:

```
Overdue tasks:

(A) Respond to P0 follow-up +G24.21.05 due:2026-02-10 [9 days overdue]
(B) Review tuition bill +V06.31.02 due:2026-02-28 [past due]
```

For each, offer: complete (if done), reschedule (set new due date), or cancel.

---

## 8. JD Integration

### 8.1 Cross-Referencing with JDex

When a task uses `+AC.ID` or `+SYS.AC.ID` project tags:

1. Look up the code in the relevant system's JDex.
2. If found, display the JDex description alongside the task when showing
   task lists.
3. If not found, warn the user: "JD code +N42.12.05 is not in the JDex.
   Create the ID, or use a different code?"

### 8.2 Context from JD System

When displaying tasks, enrich JD code references:

```
(A) Clean out garage for sale +N42.12.05 due:2026-05-01
    ↳ N42.12.05 = Garage maintenance (in P10 Personal)
```

This is informational — it helps the user remember what the JD code refers to
without switching context.

### 8.3 Task Extraction Compatibility

The inbox processor skill (§4 of jd-inbox-processor) extracts tasks in
markdown-checkbox format and appends them to `00.02 Tasks`. When both skills
are in use:

- If the task file is `todo.txt` (jdtodo.txt format), extracted tasks should
  be written in jdtodo.txt format instead of markdown checkboxes.
- The jdtodo.txt format is preferred when a `todo.txt` file exists.
- If only `00.02 Tasks.md` exists (legacy format), the inbox processor's
  format is compatible until migration (see §10).

### 8.4 Multi-System Tasks

A single `todo.txt` may reference multiple systems via `+SYS.AC.ID` tags.
Alternatively, each system may have its own `todo.txt`. Handle both approaches:

- **Single file**: All tasks in one `todo.txt`, system identified by project
  tags.
- **Per-system files**: Separate `todo.txt` per system, merged for cross-system
  views.

When displaying cross-system views, avoid duplicate `id:` confusion by
mentally namespacing — if two systems both have an `id:plan`, show the system
context to disambiguate.

---

## 9. Interaction Principles

### 9.1 When to Ask vs. When to Act

- **Act without asking**: Listing tasks, showing filtered views, resolving
  dependency graphs, enriching JD code references. These are read-only and
  safe.
- **Confirm before acting**: Completing tasks, cancelling tasks, moving tasks
  between files (todo.txt ↔ done.txt ↔ someday.txt), modifying tasks the user
  has specifically identified. Present the change and get a "yes."
- **Always ask**: Deleting task lines (prefer moving to done.txt — never
  truly delete without explicit user confirmation), modifying tasks that the
  user hasn't specifically identified, creating task files or directories.

### 9.2 Task Identification

When the user refers to a task by description ("the insurance one," "that
garage sale task"), use these signals to identify it:

1. **Keywords** in the task description
2. **JD codes** or **project tags** mentioned by the user
3. **Context tags** (`@phone`, `@laptop`)
4. **Priority** level
5. **Recency** — if the user just added or discussed a task, it's likely the
   one they mean

If ambiguous, present the top 2–3 candidates:

> I found 2 tasks matching "insurance":
>
> 1. (A) Call insurance company about claim +N42.11.03 @phone
> 2. (C) Research new homeowner's insurance quotes +N42.11.03 @laptop
>
> Which one?

### 9.3 Format Preservation

When modifying `todo.txt`:

- Preserve all existing formatting, whitespace, and ordering.
- Append new tasks at the end of the file.
- Do not re-sort the file unless the user explicitly asks.
- When removing a completed task, remove the exact line(s) including any `\`
  continuation lines.
- Preserve blank lines and any comment-like lines (though jdtodo.txt doesn't
  formally define comments, users may have them).

---

## 10. Error Handling

### 10.1 Task File Missing

If `todo.txt` doesn't exist at the expected location:

1. Check if a `00.02 Tasks/` directory exists. If so, create `todo.txt` in it.
2. If only `00.02 Tasks.md` exists (flat file), see §10.2.
3. If nothing exists, offer to create the `00.02 Tasks/` directory with an
   empty `todo.txt`.

### 10.2 Legacy Format Migration

If `00.02 Tasks.md` exists with markdown-checkbox tasks:

```markdown
- [ ] Call the insurance company
- [x] File the tax return
- [ ] Schedule dentist appointment
```

Offer to migrate to jdtodo.txt format:

1. Convert `- [ ] description` → `YYYY-MM-DD description` (using today's date
   as creation date, since the original has none).
2. Convert `- [x] description` → `x YYYY-MM-DD description` (using today as
   completion date).
3. Extract any JD references, contexts, or dates from the markdown text and
   convert to proper tags.
4. Write the converted tasks to `00.02 Tasks/todo.txt` (completed to
   `done.txt`).
5. Rename `00.02 Tasks.md` to `00.02 Tasks.md.bak` (do not delete).

Always confirm the migration plan with the user before executing.

### 10.3 Invalid JD Codes

If a task references a `+AC.ID` that isn't in the JDex:

- When listing: Show the task but flag the invalid reference.
- When adding: Warn before writing and offer alternatives.
- Do not block operations on tasks with invalid codes — they may be
  intentional placeholders.

### 10.4 Circular Dependencies

If `dep:` chains form a cycle (A depends on B, B depends on A):

- Warn the user about the cycle.
- Show which tasks are involved.
- Suggest removing one of the dependencies to break the cycle.
- Do not modify anything automatically.

### 10.5 Duplicate Task IDs

If multiple tasks in the same file have the same `id:` value:

- Warn the user when the duplicate is detected.
- Show both tasks with the duplicate ID.
- Suggest renaming one of them.

### 10.6 File Write Errors

If writing to a file fails (permissions, disk full, etc.):

- Report the error clearly.
- Do not attempt to modify the original file again.
- Show the user the change that was attempted so they can apply it manually.

---

## 11. Post-Action Summary

After any write operation (add, complete, cancel, modify), present a summary:

```
Task update complete.

P10 Personal: 12 active, 3 overdue
  Completed: 2 tasks (moved to done.txt)
  Recurring: 1 new occurrence generated
  Added: 1 new task

W20 Work: 5 active, 1 overdue
  Completed: 1 task (moved to done.txt)
```

For review sessions, provide a more detailed summary:

```
Weekly review complete.

Someday: 3 reviewed → 1 activated, 1 cancelled, 1 kept
Deferred: 2 tasks now actionable
Stale: 1 task moved to someday
Overdue: 3 tasks rescheduled
Newly unblocked: 1 task
```

---

## Quick Reference: Task File Locations

```
SYS/00-09 System/00 System management/00.02 Tasks/
├── todo.txt       # Active and deferred tasks
├── done.txt       # Completed and cancelled tasks
└── someday.txt    # @someday items (optional)
```

## Quick Reference: Task States

| State | Line prefix | Stored in |
|-------|------------|-----------|
| Incomplete | (none, or `(A)`) | todo.txt |
| Complete | `x YYYY-MM-DD` | done.txt |
| Cancelled | `~ YYYY-MM-DD` | done.txt |
| Someday | `@someday` or `due:someday` | someday.txt (optional) or todo.txt |
| Hidden | `h:1` | todo.txt (excluded from views) |

## Quick Reference: Date Tags

| Tag | Meaning | Visibility effect |
|-----|---------|-------------------|
| `due:YYYY-MM-DD` | Hard deadline | Overdue highlighting |
| `before:YYYY-MM-DD` | Soft deadline | Same as `due:` for display |
| `t:YYYY-MM-DD` | Don't show until | Hidden before date |
| `after:YYYY-MM-DD` | Can't start until | Hidden before date |
| `due:someday` | Indefinite | Hidden from default views |
