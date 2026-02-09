---
name: jd-inbox-processor
description: >
  Process a Johnny.Decimal inbox by reading items from 00.01 Inbox folders,
  classifying them against the JDex and folder structure, and routing them to
  the correct location. Use this skill whenever the user mentions processing
  their inbox, filing items, sorting their JD inbox, triaging captured notes,
  or any variation of "process my inbox," "sort my inbox," "file my stuff,"
  "what's in my inbox," or "triage my captures." Also trigger when the user
  drops files into an inbox folder and asks what to do with them, or when they
  say things like "I just dumped some stuff in my inbox" or "clean up my
  inbox." This skill handles single-system and multi-system JD setups.
---

# Johnny.Decimal Inbox Processor

This skill processes items in Johnny.Decimal `00.01 Inbox` folders — reading
each item, classifying it against the system's JDex and folder structure, and
moving it to the correct ID location. When classification is ambiguous, it asks
the user to decide. It also extracts tasks, updates the JDex, and logs every
action.

Before processing, read `references/jd-system-rules.md` for the structural
conventions that govern all classification decisions.

---

## 1. Orientation: Discover the JD Environment

Before touching any files, build a mental map of the user's JD setup.

### 1.1 Locate the JD Root

The JD root folder is wherever the user's systems live. Common locations:

- `~/Library/Mobile Documents/com~apple~CloudDocs/JD/` (iCloud Drive)
- `~/Documents/JD/`
- `~/JD/`
- A project-specific folder the user designates

If you don't know the root, ask the user. If the user says "process my inbox"
without further context, check the most common locations above. If you find
exactly one, confirm it. If you find multiple or none, ask.

### 1.2 Identify Systems

List the top-level folders under the JD root. Each folder whose name matches
the pattern `[A-Z][0-9][0-9] *` is a JD system (e.g., `N42 Personal`,
`G24 GitHub`, `S16 SixCity`).

If the user has a single system (no SYS prefix), that's fine — treat the
entire JD root as one system.

### 1.3 Load Each System's JDex

For every system you'll process, read the JDex file at:

```
SYS/00-09 */00 */00.00 *JDex*
```

The JDex is the authoritative index of every area, category, and ID. It is
your primary classification reference. If the JDex is missing or empty for a
system, fall back to reading the folder structure directly — but flag this to
the user as something that should be fixed.

### 1.4 Snapshot the Folder Structure

For each system, list the area folders, category folders, and ID folders to
depth 3. This gives you the physical layout to validate against the JDex and
to discover any folders not yet in the index.

---

## 2. Inbox Discovery

### 2.1 Locate Inboxes

Each system's inbox lives at:

```
SYS/00-09 */00 */00.01 Inbox/
```

List the contents of every inbox you find. If the user asked to process a
specific system's inbox (e.g., "process my N42 inbox"), limit to that one.

### 2.2 Assess the Inbox

For each inbox, report to the user:

- **Item count**: How many files/folders are in the inbox
- **Item types**: File types present (`.md`, `.pdf`, `.jpg`, `.txt`, etc.)
- **Date range**: Earliest and latest date-prefixed items

If the inbox is empty, say so and move on. If it contains more than ~20 items,
suggest processing in batches and ask the user how many to tackle.

---

## 3. Process Each Item

Work through inbox items one at a time (or in logical groups if items are
clearly related, like a note and its accompanying photo). For each item:

### 3.1 Read and Understand the Item

- **Text files** (`.md`, `.txt`): Read the full content.
- **PDFs**: Read if possible; if not readable, use the filename and any
  available metadata.
- **Images** (`.jpg`, `.png`): Describe what you see. If it's a photo of a
  document, extract what you can.
- **Other files**: Use the filename, extension, and file size to infer purpose.

Summarize what the item is about in one sentence. This is your "item summary"
and is used in the processing log.

### 3.2 Classify the Item

Classification follows a strict decision tree. At each level, you're choosing
from a bounded set (≤10 options). This is where the JDex is critical.

Read `references/classification-heuristics.md` for detailed guidance on
ambiguous cases.

**Step 1 — Which system?** (Multi-system setups only)

If the user has multiple systems, determine which system the item belongs to.
Signals include:

- Explicit system references in the content ("SixCity," "work," "personal")
- Content domain (tax docs → personal, buyer meeting notes → work)
- Filename patterns or prefixes the user may use

If ambiguous between systems, ask the user.

**Step 2 — Which area?**

Read the area list from the JDex. Choose the area whose description best
matches the item's content. Areas are broad domains (`10-19 Home & property`,
`30-39 Money & legal`, etc.).

If no area fits, this may indicate:

- The item belongs in a different system
- The system needs a new area (rare — flag for user decision, do not create)
- The item is miscategorized in the user's mind (ask)

**Step 3 — Which category?**

Within the chosen area, pick the category. Categories are collections of
similar things. This is "where you work" — the most important classification
decision.

**Step 4 — Which ID?**

Within the category, determine the specific ID folder:

- **Existing ID matches**: The item clearly belongs in an existing ID folder.
  Use it.
- **+SUB match**: If the category uses `+SUB` extensions (e.g., `11.01+BOBS`),
  match the item to the correct sub-entry.
- **New ID needed**: If no existing ID fits, you may need to create one.
  See §3.4.

### 3.3 Confidence Assessment

After classification, assess your confidence:

- **High confidence** (clear match, unambiguous): Proceed to filing.
- **Medium confidence** (reasonable match, but another location is plausible):
  State your recommendation and the alternative, then ask the user to confirm.
- **Low confidence** (genuinely unclear): Present the top 2–3 candidate
  locations with brief reasoning, and ask the user to choose.

The threshold: if you'd bet money on the classification, it's high confidence.
If you'd want a second opinion, it's medium. If you're guessing, it's low.

When asking the user, be specific. Not "where should this go?" but rather:

> This looks like a health insurance claim. I'd file it at **N42.34.01**
> (Health insurance). Does that sound right, or does it belong somewhere else?

### 3.4 Creating New IDs

If an item needs a new ID (no existing ID in the category fits), follow these
rules strictly:

1. **Never create a new area or category without explicit user approval.** These
   are structural decisions that affect the whole system. Flag the need and ask.
2. **New IDs within an existing category are lower-stakes.** You may propose a
   new ID, but always confirm with the user before creating it.
3. **Determine the next sequential ID number.** Look at existing IDs in the
   category. If the highest is `15.23`, the next is `15.24`. If the category
   uses standard zeros (`.00`–`.09` reserved), the first content ID is `.10`
   or the next available number above `.09`.
4. **Name the new ID clearly.** Follow the pattern of existing IDs in the
   category. JD ID folder names are: `AC.ID Description` (e.g.,
   `15.24 Trip to Portland`).
5. **After creating the folder, update the JDex** with the new entry.

For +SUB entries, the process is similar but uses the `+NNNN` or `+CODE`
pattern established in that category.

### 3.5 Filing the Item

Once the destination is confirmed:

1. **Rename the file** with a date prefix if it doesn't already have one.
   Format: `YYYY-MM-DD Description.ext`. Use today's date if the item has no
   inherent date. Use the item's date if it does (e.g., a receipt dated
   2026-01-15).

2. **Move the file** from the inbox to the destination ID folder.

3. **If the item contains actionable tasks**, extract them. See §4.

4. **If the item is both a record and a task source**, file the record AND
   extract the tasks. Don't choose one or the other.

### 3.6 Items That Don't Belong

Some inbox items are:

- **Stale or irrelevant**: Old notes that are no longer needed. Confirm with
  the user, then move to `00.09 Archive` if that folder exists, or delete if
  the user prefers.
- **Duplicates**: The same content already exists at the destination. Flag to
  the user and offer to delete the inbox copy.
- **Cross-system references**: The item belongs in System A but was captured
  in System B's inbox. Move it to System A's inbox (or directly to the correct
  ID if classification is clear), and note the cross-reference.

---

## 4. Task Extraction

Many inbox items contain embedded tasks — things the user needs to do. When
you identify a task, extract it.

### 4.1 Identifying Tasks

Look for:

- Explicit action items ("need to send pricing," "follow up with Sarah")
- Deadlines ("by end of week," "before March 8")
- Commitments ("promised to deliver," "agreed to check on")
- Questions that need answers ("check with warehouse," "ask about delivery")

### 4.2 Task Format

Append extracted tasks to the system's task file at:

```
SYS/00-09 */00 */00.02 Tasks*
```

Each task should include:

```markdown
- [ ] [Task description]
      → [JD reference: AC.ID or AC.ID+SUB] | Deadline: [date or "none"] | Source: [inbox item filename]
```

If the task file doesn't exist, create it. If the task file uses a different
format, match the existing format.

### 4.3 Task Context

When possible, add context from the JD system to the task:

- Reference relevant IDs (e.g., "Pricing info in 31.03")
- Note related contacts or accounts
- Link to the filed record

---

## 5. Logging

After processing each item, append a log entry to:

```
SYS/00-09 */00 */00.03 Processing log.md
```

If this file doesn't exist, create it.

### Log Entry Format

```markdown
### YYYY-MM-DD HH:MM — Inbox Processing

| Item | Action | Destination | Notes |
|------|--------|-------------|-------|
| `filename.ext` | Filed | `AC.ID Description` | One-line summary |
| `filename2.md` | Filed + Task | `AC.ID+SUB` | Task added to 00.02 |
| `filename3.pdf` | Needs review | `00.04 Needs review` | Ambiguous; user asked to decide later |
| `filename4.txt` | Archived | `00.09 Archive` | Stale item, confirmed by user |

**Items processed**: N
**Tasks extracted**: N
**Needs review**: N
**New IDs created**: [list, or "none"]
```

---

## 6. JDex Maintenance

After all items are processed, verify the JDex:

1. **New entries**: If any new IDs or +SUB entries were created during
   processing, confirm they've been added to the JDex.
2. **Orphan check** (optional, suggest to user): Compare the JDex against the
   actual folder structure. Flag any folder that exists but isn't in the JDex,
   and any JDex entry whose folder doesn't exist.

---

## 7. Multi-System Processing

When the user says "process my inboxes" (plural) or doesn't specify a system:

1. List all systems with non-empty inboxes.
2. Report the item count per system.
3. Ask whether to process all of them or just specific ones.
4. Process each system's inbox in sequence. Keep system context separate —
   don't confuse N42's JDex with G24's categories.
5. At the end, give a combined summary across all systems.

---

## 8. Interaction Principles

### 8.1 When to Ask vs. When to Act

- **Act without asking**: Filing an item to a clearly matching existing ID with
  proper date-prefix naming. This is routine and unambiguous.
- **State and confirm in batch**: When processing multiple items, you can
  propose a batch of filings and let the user approve them all at once rather
  than one at a time. Present a table: item → proposed destination.
- **Ask before acting**: Creating new IDs, creating new +SUB entries, archiving
  or deleting items, moving items between systems, any classification with
  medium or low confidence.
- **Always ask**: Creating new categories or areas. These are structural changes.

### 8.2 Batch Processing Mode

For inboxes with many items, offer batch mode:

1. Read all items.
2. Classify each one.
3. Present a table of proposed actions.
4. Let the user approve, modify, or flag specific items.
5. Execute all approved actions.
6. Handle flagged items individually.

This is faster than one-at-a-time for large inboxes while still giving the
user control.

### 8.3 Learning from User Decisions

When the user corrects a classification or makes a non-obvious filing decision,
note the reasoning. This helps you classify similar items in the future within
the same session. For example, if the user says "actually, I keep all
Streetside stuff in 31.02, not by account," apply that pattern to subsequent
Streetside items.

---

## 9. Error Handling

- **JDex missing**: Warn the user. Offer to generate one from the folder
  structure. Process using folders as the reference, but note that a JDex
  should be created.
- **Inbox folder missing**: A system without `00.01 Inbox` has no inbox to
  process. Note this and move on.
- **Unreadable files**: If a file can't be read (binary, corrupted, encrypted),
  log it, skip it, and tell the user.
- **Filesystem errors**: If a move fails (permissions, path issues), report the
  error, skip the item, and continue with the rest.
- **JDex/folder mismatch**: If the JDex says an ID exists but the folder
  doesn't (or vice versa), flag the inconsistency. Don't silently create or
  delete anything to "fix" it — the user decides.

---

## 10. Post-Processing Summary

After completing all inbox processing, present a summary:

```
Inbox processing complete.

[System name]: [N] items processed
  → [N] filed to existing IDs
  → [N] tasks extracted to 00.02
  → [N] new IDs created: [list]
  → [N] items need review (in 00.04)
  → [N] archived

[Repeat for each system processed]

JDex status: [up to date / N entries added / needs audit]
```

If any items were moved to "needs review," remind the user they're waiting.

---

## Quick Reference: File Naming Convention

All files in JD ID folders use date-prefix naming:

```
YYYY-MM-DD Description.ext
```

- Use the item's inherent date when available (document date, receipt date,
  meeting date).
- Use the capture/creation date if no inherent date exists.
- Use today's date as a last resort.
- Keep descriptions short but specific. Match the style of existing files in
  the same folder.
- No special characters in filenames beyond hyphens and spaces.

---

## Quick Reference: Standard Zeros

These reserved IDs exist (or may exist) in each category:

| ID | Purpose |
|----|---------|
| `.00` | Category JDex / index |
| `.01` | Inbox |
| `.02` | Tasks / action items |
| `.03` | Templates (or processing log in system `00`) |
| `.04` | Links / quick reference (or needs-review in system `00`) |
| `.08` | Someday / parking lot |
| `.09` | Archive |

Not all systems use all standard zeros. Check what actually exists.
