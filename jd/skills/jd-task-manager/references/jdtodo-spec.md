# jdtodo.txt Format Specification

**Version 0.1.0** · February 2026

A plain-text task format that extends [todo.txt](https://github.com/todotxt/todo.txt) with [Johnny.Decimal](https://johnnydecimal.com/) integration, enriched date semantics, task relationships, and inline notes.

---

## Design principles

1. **A single task is a single logical line.** Backslash continuation allows multiline notes while preserving the one-task-one-entry principle.
2. **Human-readable without tooling.** The file must make sense in any text editor.
3. **Sortable by standard tools.** Priority and dates appear at line start so `sort` works.
4. **Backward-compatible.** Every valid todo.txt file is a valid jdtodo.txt file. Extensions use the established `key:value` mechanism.
5. **JD-native.** Johnny.Decimal codes are first-class citizens in the project tag namespace.

---

## 1. Incomplete tasks

An incomplete task is any line that does not begin with `x ` or `~ ` and is not a continuation line (see §7).

### 1.1 Priority

If present, priority ALWAYS appears first. It is an uppercase letter `A`–`Z` in parentheses followed by a space.

```
(A) Call the insurance company +N42.11.03 @phone
```

### 1.2 Creation date

If present, the creation date appears directly after priority (and a space), or first if no priority exists. Format: `YYYY-MM-DD`.

```
(A) 2026-02-09 Call the insurance company +N42.11.03 @phone
2026-02-09 Pick up dry cleaning @errands
```

### 1.3 Contexts

A context is preceded by a single space and `@`. It contains any non-whitespace characters. A task may have zero or more contexts.

```
(B) Review Q3 budget spreadsheet @laptop @office
```

Contexts are unchanged from the todo.txt specification. The following GTD-aligned contexts are recommended but not required:

| Context | Semantic meaning |
|---|---|
| `@waiting` | Blocked on an external person or event |
| `@someday` | Not currently actionable; review periodically |
| `@agenda` | Discuss at next meeting with the relevant person |

### 1.4 Projects and JD codes

A project is preceded by a single space and `+`. It contains any non-whitespace characters. A task may have zero or more projects.

**Johnny.Decimal codes are valid project tags.** The `+` prefix followed by a JD address creates a direct link between a task and its place in a Johnny.Decimal filing system.

#### Supported JD project formats

| Format | Example | Meaning |
|---|---|---|
| Named project | `+GarageSale` | Traditional todo.txt project |
| AC.ID | `+21.03` | Single-system JD reference |
| SYS.AC.ID | `+N42.21.03` | Multi-system JD reference |
| AC.ID+SUB | `+21.03+0012` | JD reference with extend-the-end |
| SYS.AC.ID+SUB | `+N42.21.03+0012` | Full multi-system + extended reference |
| Category only | `+N42.21` | Reference to an entire JD category |

```
(A) 2026-02-09 Call Goodwill to schedule pickup +N42.12.05 +GarageSale @phone
(B) File Vanessa's tuition receipt +V06.31.02 @laptop
Review brewery portfolio notes +S16.32.01 @office
```

A task may combine named projects and JD codes freely. JD codes on a task indicate where the task's *reference material* lives, not where the task itself is stored. The task file (`todo.txt`) may live anywhere; the JD code is a cross-reference.

**Regex for JD project tags:**

```
\+([A-Z]\d{2}\.)?\d{2}\.\d{2}(\+[A-Za-z0-9]+)?
```

This matches `+AC.ID`, `+SYS.AC.ID`, `+AC.ID+SUB`, and `+SYS.AC.ID+SUB`.

---

## 2. Complete tasks

### 2.1 Completion marker

A completed task starts with a lowercase `x` followed by a space.

```
x 2026-02-09 2026-02-01 Call Goodwill to schedule pickup +N42.12.05 @phone
```

### 2.2 Completion date

The completion date appears directly after `x `, in `YYYY-MM-DD` format. If a creation date was present, it follows the completion date.

```
x 2026-02-09 2026-02-01 File insurance claim +N42.11.03
  ^completed  ^created
```

### 2.3 Preserved priority

When a task is completed, clients typically strip the priority. To preserve it for reporting or filtering, use the `pri:` key:

```
x 2026-02-09 2026-02-01 Call Goodwill +GarageSale @phone pri:A
```

### 2.4 Cancelled tasks

A cancelled task starts with a tilde `~` followed by a space. It uses the same date format as completed tasks.

```
~ 2026-02-09 2026-02-01 Order extra tables for sale +GarageSale
  ^cancelled  ^created
```

Cancelled tasks sort after completed tasks (`x` < `~` in ASCII when lowercase letters precede tilde — but both sort below active tasks that start with `(` or digits). Clients SHOULD support filtering cancelled tasks separately from completed ones.

**Rationale:** The todo.txt community has no standard for cancelled tasks, yet any serious task system needs to distinguish "done" from "abandoned." The `~` marker is visually distinct, sorts predictably, and preserves the line-oriented format.

---

## 3. Date extensions

jdtodo.txt adopts two de facto community standards and adds natural-language date semantics.

### 3.1 Due date — `due:YYYY-MM-DD`

The date by which the task must be completed. Clients SHOULD highlight overdue tasks.

```
(A) Submit tax return +N42.13.02 due:2026-04-15
```

### 3.2 Threshold date — `t:YYYY-MM-DD`

The date before which the task should be hidden from default views. This creates a "don't show until" behavior, equivalent to GTD's tickler file.

```
(B) Renew car registration +N42.12.01 t:2026-06-01 due:2026-06-30
```

A task with `t:2026-06-01` is invisible in standard views until June 1, 2026. The gap between `t:` and `due:` defines the task's working window.

### 3.3 Natural-language date aliases

Real-world tasks often have fuzzy temporal constraints that don't fit a single `due:` date. jdtodo.txt provides two additional date tags and a symbolic value to capture these semantics.

#### `before:YYYY-MM-DD`

The task should be completed before this date. Semantically equivalent to `due:` but expresses a softer deadline — a target rather than a hard cutoff.

```
Get the garage cleaned out before the party +GarageSale before:2026-05-01
```

Clients MAY treat `before:` identically to `due:` for sorting and highlighting purposes. The distinction is semantic: `due:` is a hard deadline, `before:` is a preference.

#### `after:YYYY-MM-DD`

The task cannot or should not begin until after this date. Semantically equivalent to `t:` but expresses a dependency on an external event rather than a deferral decision.

```
Schedule post-surgery follow-up +N42.25.01 after:2026-03-15
```

Clients MAY treat `after:` identically to `t:` for visibility filtering. The distinction is semantic: `t:` means "I'm choosing to defer this," while `after:` means "this isn't possible until then."

#### The `someday` value

Any date field may use the literal value `someday` instead of a date to indicate an indefinite future timeframe.

```
Learn to play banjo +N42.41.08 due:someday
Rebuild the front porch +N42.12.05 after:someday
```

Tasks with `due:someday` SHOULD be treated equivalently to the `@someday` context for filtering purposes. Clients SHOULD hide `someday` tasks from default views.

### 3.4 Date field summary

| Tag | Meaning | Hard/Soft | Equivalent to |
|---|---|---|---|
| `due:` | Must be done by this date | Hard deadline | — |
| `before:` | Should be done by this date | Soft deadline | Softer `due:` |
| `t:` | Don't show until this date (user deferral) | Visibility gate | — |
| `after:` | Can't start until this date (external constraint) | Visibility gate | Contextual `t:` |

All date values are `YYYY-MM-DD` or the literal `someday`.

**Combining dates:** A task may use multiple date tags to define a complete temporal window:

```
(B) File quarterly sales report +G24.31.02 after:2026-03-31 due:2026-04-15
```

This task becomes actionable on March 31 and must be completed by April 15.

---

## 4. Task relationships

jdtodo.txt uses three `key:value` tags for task identity and relationships. Identifiers are short, non-whitespace strings without colons. Human-readable identifiers are encouraged.

### 4.1 Task identity — `id:IDENTIFIER`

Assigns a stable, unique identifier to a task. This identifier is referenced by other tasks' `dep:` and `sup:` tags.

```
(A) Plan the garage sale +GarageSale id:plan-sale
```

Identifiers MUST be unique within a single `todo.txt` file. They SHOULD be short, descriptive, and free of special characters beyond hyphens and underscores. Tooling MAY auto-generate identifiers (e.g., `id:xq3`).

### 4.2 Dependency — `dep:IDENTIFIER`

Declares that this task is **blocked by** another task. The referenced task must be completed before this task becomes actionable.

```
(B) Post signs around neighborhood +GarageSale dep:plan-sale
(B) Price all items in garage +GarageSale dep:plan-sale
```

Both tasks are blocked until `id:plan-sale` is completed. Clients SHOULD hide or de-emphasize tasks with unresolved dependencies. A task MAY have multiple `dep:` tags.

### 4.3 Supertask — `sup:IDENTIFIER`

Declares that this task is a **subtask of** another task. This expresses decomposition ("this is part of that") rather than sequencing ("this must come after that").

```
(A) Clean out the garage +GarageSale id:garage-clean
(B) Sort items into keep/sell/donate +GarageSale sup:garage-clean
(B) Sweep and mop garage floor +GarageSale sup:garage-clean
(C) Arrange sale items on tables +GarageSale sup:garage-clean dep:sort-items
```

`sup:` does NOT imply blocking. All subtasks of a parent can be worked in parallel unless `dep:` adds explicit ordering. A task MAY have both `sup:` and `dep:` tags.

### 4.4 Relationship summary

| Tag | Direction | Meaning | Implies blocking? |
|---|---|---|---|
| `id:X` | Self | "I am X" | — |
| `dep:X` | → X | "I am blocked by X" | Yes |
| `sup:X` | → X | "I am a subtask of X" | No |

**Example: a complete task graph**

```
(A) Plan the garage sale +GarageSale id:plan-sale due:2026-04-01
(B) Post signs around neighborhood +GarageSale id:post-signs dep:plan-sale
(B) Sort items into keep/sell/donate +GarageSale id:sort-items dep:plan-sale
(C) Price all sale items +GarageSale dep:sort-items
(B) Set up tables in driveway +GarageSale dep:plan-sale due:2026-05-01
```

---

## 5. Recurrence — `rec:[+]N[d|w|m|y|b]`

A recurring task regenerates itself when completed. The `rec:` tag specifies the interval.

### 5.1 Interval units

| Code | Unit |
|---|---|
| `d` | Days |
| `w` | Weeks |
| `m` | Months |
| `y` | Years |
| `b` | Business days (Mon–Fri) |

### 5.2 Strict vs. non-strict

Without a `+` prefix, the next occurrence is calculated from the **completion date**:

```
Water the plants rec:3d due:2026-02-09
# Completed Feb 12 → new task due:2026-02-15 (12 + 3)
```

With a `+` prefix, the next occurrence is calculated from the **original due date**:

```
Pay rent rec:+1m due:2026-02-01
# Completed Feb 1 → new task due:2026-03-01 (Feb 1 + 1m)
```

Use strict (`+`) for calendar-anchored obligations. Use non-strict for tasks where the gap since last completion matters.

### 5.3 Recurrence behavior

When a recurring task is completed:

1. The original task is marked complete (`x YYYY-MM-DD ...`).
2. A new incomplete task is created with the same text, projects, contexts, and tags.
3. `due:` and `t:` are shifted forward by the recurrence interval. The gap between `t:` and `due:` is preserved.
4. `before:` and `after:` are shifted forward by the same interval.
5. The creation date on the new task is set to the completion date.

---

## 6. Hidden tasks — `h:1`

A task with `h:1` is hidden from all default views. Its primary purpose is to preserve project and context tags in autocomplete/filter pickers without cluttering the task list.

```
+GarageSale @phone @errands h:1
```

This line ensures `+GarageSale`, `@phone`, and `@errands` appear in filter pickers even when no active tasks use them. Clients MUST exclude `h:1` tasks from standard views and counts.

---

## 7. Notes

A task may include a freeform text note separated from the task body by ` --- ` (space, three hyphens, space). For notes that span multiple lines, a trailing backslash (`\`) at the end of a line indicates that the next line continues the note.

### 7.1 Inline notes

```
(A) Call insurance company about claim +N42.11.03 @phone --- Ask about deductible and reimbursement timeline.
(B) Schedule vet appointment for Max @phone --- Last checkup was October 2025. Needs rabies booster.
```

The note is everything after the ` --- ` separator to the end of the line.

### 7.2 Multiline notes

A trailing `\` (backslash as the very last character on the line, with no trailing whitespace) indicates that the next line is a continuation of the note. The continuation line's leading whitespace is trimmed — indentation is cosmetic and does not affect the parsed note content.

```
(B) Schedule vet appointment for Max @phone --- Last checkup was October 2025. \
    Needs rabies booster. \
    Also ask about the limping issue.
```

This task has a three-line note:

```
Last checkup was October 2025.
Needs rabies booster.
Also ask about the limping issue.
```

Continuation lines MAY be indented for readability. The following are equivalent:

```
Task description --- First line of note. \
Second line of note.

Task description --- First line of note. \
    Second line of note.
```

### 7.3 Rules

1. The note separator is ` --- ` — at least one space before and after three hyphens. The separator MUST NOT appear at the start of a line.
2. Everything before the first ` --- ` on a task line is the task body (parsed for priority, dates, contexts, projects, and key:value pairs). Everything after it is the note (freeform text, not parsed for task metadata).
3. A trailing `\` must be the final character on the line (no whitespace after it). It signals that the next physical line is appended to the note with a newline character.
4. Leading whitespace on continuation lines is trimmed. A continuation line that is entirely whitespace (after trimming) produces a blank line in the note.
5. Continuation lines MUST NOT be parsed as tasks. A line that follows a `\`-terminated line is always a note continuation, regardless of its content.
6. Key:value pairs (like `due:` or `id:`) MUST appear in the task body (before ` --- `), never in the note. Clients MUST NOT parse the note text for task metadata.
7. A task with no ` --- ` separator has no note. The separator is not required.
8. Clients SHOULD display notes as secondary text beneath or beside their parent task. Clients MAY collapse notes by default.

### 7.4 Notes on completed and cancelled tasks

Notes follow their parent task through state changes:

```
x 2026-02-09 2026-02-01 Call insurance company +N42.11.03 @phone --- Asked about deductible. $500, reimbursement in 2-3 weeks.
~ 2026-02-08 2026-02-01 Order extra folding tables +GarageSale --- Cancelled — Lauren has enough tables.
```

Notes on completed tasks serve as a lightweight activity log. Notes on cancelled tasks record the reason for cancellation.

### 7.5 Sorting behavior

Standard `sort` operates on complete physical lines. Since the task body (priority, dates) appears before the ` --- ` separator, sorting by line content produces correct task ordering. Multiline notes (using `\` continuation) will cause continuation lines to sort independently — clients that support sorting SHOULD treat a task and its continuation lines as a single logical unit.

---

## 8. JD-specific conventions

### 8.1 Task file location

A `todo.txt` file MAY live at the standard zeros location within a JD system:

```
SYS/00-09 System/00 System management/00.02 Tasks/todo.txt
```

For multi-system users, each system MAY have its own `todo.txt`:

```
N42/00-09 System/00 System management/00.02 Tasks/todo.txt
G24/00-09 System/00 System management/00.02 Tasks/todo.txt
```

Alternatively, a single `todo.txt` MAY serve all systems, using `+SYS.AC.ID` project tags to indicate system affiliation.

### 8.2 Done file

Completed and cancelled tasks SHOULD be moved to a `done.txt` file in the same directory. This keeps the active `todo.txt` lean while preserving history.

```
00.02 Tasks/
├── todo.txt      # Active and deferred tasks
└── done.txt      # Completed and cancelled tasks
```

### 8.3 Someday file

Tasks tagged `@someday` or with `due:someday` MAY optionally be stored in a separate `someday.txt` file, aligning with the JD standard zeros scheme where `xx.08` is reserved for "someday" items.

```
00.02 Tasks/
├── todo.txt
├── done.txt
└── someday.txt
```

### 8.4 Cross-system task references

When a task in one system references material in another, use the full `SYS.AC.ID` format in the project tag:

```
(B) Review Vanessa's health insurance options +V06.32.01 +N42.13.05 @laptop
```

This task lives in whichever `todo.txt` the user chooses, but references material in both the V06 and N42 filing systems.

---

## 9. Complete grammar

### 9.1 Task line (ABNF-style)

```
logical-line  = task-body [" --- " note-body]
task-body     = [completion] [priority] [creation-date] description
completion    = ("x" / "~") SP date SP
priority      = "(" ALPHA ")" SP
creation-date = date SP
description   = 1*(word / context / project / kv-pair)

note-body     = note-text *("\" LF note-cont)
note-cont     = *WSP note-text          ; leading whitespace trimmed

context       = SP "@" non-ws+
project       = SP "+" non-ws+
kv-pair       = SP key ":" value
key           = non-ws-no-colon+
value         = non-ws+
date          = YYYY "-" MM "-" DD
```

### 9.2 Note separator

```
separator   = SP "---" SP              ; exactly " --- " (space-hyphen-hyphen-hyphen-space)
```

### 9.3 Recognized key:value pairs

| Key | Value format | Required? | Spec section |
|---|---|---|---|
| `due` | `YYYY-MM-DD` or `someday` | Optional | §3.1 |
| `before` | `YYYY-MM-DD` or `someday` | Optional | §3.3 |
| `t` | `YYYY-MM-DD` or `someday` | Optional | §3.2 |
| `after` | `YYYY-MM-DD` or `someday` | Optional | §3.3 |
| `id` | Non-whitespace, no colons | Optional | §4.1 |
| `dep` | Non-whitespace, no colons | Optional | §4.2 |
| `sup` | Non-whitespace, no colons | Optional | §4.3 |
| `rec` | `[+]N[d\|w\|m\|y\|b]` | Optional | §5 |
| `h` | `1` | Optional | §6 |
| `pri` | `A`–`Z` | Optional | §2.3 |

Additional `key:value` pairs are permitted. Clients SHOULD preserve unrecognized pairs.

---

## 10. Complete example

```
(A) 2026-02-09 Submit quarterly tax estimate +N42.13.02 @laptop due:2026-04-15 t:2026-03-15 --- Accountant said to use last year's numbers as a baseline. \
    Need 1040-ES form from IRS website.
(A) 2026-02-09 Respond to P0 incident follow-up +G24.21.05 @work due:2026-02-10
(B) 2026-02-09 Schedule dentist appointment for Hannah +H11.25.01 @phone before:2026-03-01
(B) 2026-02-09 Review Vanessa's spring tuition bill +V06.31.02 +N42.13.05 @laptop due:2026-02-28 --- Vanessa said there's a new lab fee this semester.
(C) 2026-02-09 Research new homeowner's insurance quotes +N42.11.03 @laptop after:2026-06-01 --- Current policy renews in July. Start shopping 30 days before.
(C) 2026-02-09 Plan summer camping trip +N42.41.05 @home due:someday
(B) 2026-02-09 Water all houseplants @home rec:3d due:2026-02-12
(A) 2026-02-09 Clean out garage for sale +GarageSale +N42.12.05 id:garage-clean due:2026-05-01
(B) 2026-02-09 Sort items into keep/sell/donate +GarageSale sup:garage-clean id:sort-items dep:garage-clean
(B) 2026-02-09 Post signs around neighborhood +GarageSale dep:garage-clean
(C) 2026-02-09 Price all sale items +GarageSale dep:sort-items
x 2026-02-09 2026-02-01 Call Goodwill to schedule pickup +GarageSale @phone pri:A --- Scheduled for Feb 15. Confirmation #GW-20260215-003.
~ 2026-02-08 2026-02-01 Order extra folding tables +GarageSale --- Cancelled — Lauren has enough tables at Thorpe house.
```

---

## 11. Compatibility

### 11.1 With standard todo.txt

Every valid todo.txt file is a valid jdtodo.txt file. The following extensions are additive and do not conflict with the base specification:

- `due:`, `before:`, `t:`, `after:` use the standard `key:value` mechanism
- `id:`, `dep:`, `sup:` use the standard `key:value` mechanism
- `rec:`, `h:`, `pri:` use the standard `key:value` mechanism
- JD codes in `+project` tags use only characters already valid in project names
- `~` cancelled marker occupies the same structural position as `x`
- ` --- ` note separator appears inline; unsupported clients display the note as part of the task text (graceful degradation)
- `\` continuation lines appear as separate lines in unsupported clients (partial degradation for multiline notes)

### 11.2 With existing todo.txt clients

| Feature | Compatible clients | Graceful degradation |
|---|---|---|
| `due:`, `t:`, `rec:`, `h:1` | Sleek, topydo, SimpleTask, SwiftoDo, TodoTxtMac | Full support |
| `id:`, `dep:`, `sup:` | topydo (uses `id:`/`p:` natively) | Other clients preserve tags but ignore semantics |
| `before:`, `after:` | None natively | Preserved as opaque `key:value` pairs |
| `due:someday` | None natively | Preserved as opaque value |
| `~` cancelled | None natively | Lines ignored or treated as active tasks |
| ` --- ` inline notes | None natively | Note text appears as part of task description (readable) |
| `\` continuation | None natively | Continuation lines appear as separate (malformed) lines |
| `+SYS.AC.ID` | All (treated as project tag) | Filtered and grouped like any project |

---

## Acknowledgments

This specification builds on the [todo.txt format](https://github.com/todotxt/todo.txt) created by Gina Trapani, the [Johnny.Decimal system](https://johnnydecimal.com/) created by Johnny Noble, and de facto community extensions pioneered by [topydo](https://github.com/topydo/topydo), [SimpleTask](https://github.com/mpcjanssen/simpletask-android), [Sleek](https://github.com/ransome1/sleek), and [SwiftoDo](https://swiftodoapp.com/).
