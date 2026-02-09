# Johnny.Decimal System Rules Reference

Read this file to understand the structural conventions that govern all
classification and filing decisions during inbox processing.

---

## The Three-Level Hierarchy

Johnny.Decimal has exactly three levels. Nothing deeper.

```
Area (10-19, 20-29, ..., 90-99)
  └── Category (two-digit number within the area range: 11, 12, ..., 19)
        └── ID (AC.ID format: 11.01, 11.02, ..., 11.99)
```

### Areas

- Represented as ranges of ten: `10-19`, `20-29`, ..., `90-99`
- `00-09` is reserved for system management
- Maximum 10 areas per system (including `00-09`)
- Areas are the broadest groupings — major life/work domains
- Folder name format: `AC-range Description` (e.g., `10-19 Home & property`)
- **Never save files directly in area folders.** They are structural only.

### Categories

- Two-digit numbers within an area's range (e.g., `11`, `12`, ..., `19` for
  area `10-19`)
- Maximum 10 categories per area
- Categories are "where you work" — the most important organizational unit
- Folder name format: `AC Description` (e.g., `11 House maintenance`)
- **Never save files directly in category folders.** They are structural only.

### IDs

- Format: `AC.ID` — always two digits, decimal point, two digits
- Example: `11.23`, `34.01`, `72.15`
- Sequential within a category, starting at `.01` (or `.10` if standard
  zeros are used for `.00`–`.09`)
- Maximum 99 IDs per category (`.01` through `.99`)
- Folder name format: `AC.ID Description` (e.g., `11.23 Furnace maintenance`)
- **IDs are where actual content lives.** All files go inside ID folders.

---

## The AC.ID Notation

The five-character address `AC.ID` tells you everything:

- First digit → Area (e.g., `1` → area `10-19`)
- First two digits → Category (e.g., `15` → category 15)
- Digits after the decimal → Item number within the category

Example: `15.52` means area `10-19`, category `15`, item 52.

---

## Multi-System Notation: SYS.AC.ID

When multiple systems exist, a three-character system identifier prefixes the
address: `[A-Z][0-9][0-9]`.

- Examples: `P10.15.52`, `W20.11.01`, `C40.22.03`
- The system prefix is part of the folder name: `P10 Personal/`, `W20 Work/`
- 2,600 possible system codes (A00–Z99)
- Each system has its own independent area/category/ID namespace

---

## Standard Zeros

Reserved IDs at each level for system management:

### System Level (Area 00-09, Category 00)

| ID | Purpose |
|----|---------|
| `00.00` | Master JDex (the system's complete index) |
| `00.01` | **Inbox** — the unsorted capture bucket |
| `00.02` | Tasks / action items / follow-ups |
| `00.03` | Templates, or processing log |
| `00.04` | Links / quick reference, or needs-review queue |
| `00.08` | Someday / parking lot for non-urgent ideas |
| `00.09` | Archive for completed or stale items |

### Category Level

Individual categories may also use `.00`–`.09` for their own management
(category index, inbox, etc.), but this is less common and varies by
implementation.

---

## The +SUB Extension: AC.ID+SUB

For items that repeat across many instances within a single ID:

### Numeric +SUB

```
AC.ID+NNNN
```

Zero-padded four-digit sequential number. Example:

```
53.01+0001  weather-api
53.01+0002  task-runner
53.01+0003  auth-dashboard
```

### Named +SUB

```
AC.ID+CODE
```

Short alphanumeric code. Example:

```
11.01+REDD  The Red Door
11.01+OAKS  Oak Street Cafe
11.01+RIVR  Riverside Grill
```

### When to Use +SUB vs. Sequential IDs

| Use +SUB when... | Use sequential IDs when... |
|---|---|
| Many instances of the same kind of thing | Different kinds of things in a category |
| Expect 20+ items over the system's lifetime | Expect fewer than 20 items |
| Items are named/titled | Items are primarily date-driven documents |

---

## File Naming Within IDs

All files inside ID folders use date-prefix naming:

```
YYYY-MM-DD Description.ext
```

Rules:
- Use the document's inherent date when available
- Use the capture/creation date if no inherent date
- Use today's date as a last resort
- Descriptions should be short, specific, and consistent with the folder's
  other files
- No special characters beyond hyphens and spaces

---

## The JDex

The JDex (Johnny.Decimal Index) is the master registry of every ID in a
system. It is the single source of truth.

Key properties:
- Stored as a Markdown file at `00.00 JDex.md` (or similar)
- Lists every area, category, and ID with a one-line description
- Prevents duplicate IDs
- Records cross-platform and cross-system references
- May include +SUB entries inline

When classifying inbox items, **always consult the JDex first**, then validate
against the folder structure. If they disagree, the JDex is authoritative for
what *should* exist; the filesystem is authoritative for what *does* exist.

---

## Cross-System References

When an ID in one system points to content in another system, the convention
is a `_reference.md` file in the referencing ID folder:

```markdown
# Reference

This ID references content in another system.

- **Source:** P10.34.01 (Health insurance)
- **Location:** P10 Personal / 30-39 Money & legal / 34 Insurance / 34.01
- **Notes:** [context about why this reference exists]
```

During inbox processing, if an item belongs in a different system, either:
1. Move it directly to the correct system and ID, or
2. Move it to the correct system's inbox for processing in that system's context

---

## Core Philosophy (Relevant to Classification)

These principles should guide ambiguous classification decisions:

1. **"Decide and document."** When an item could go in two places, pick one
   and record the decision. The JDex handles ambiguity — not the hierarchy.

2. **Compression over granularity.** One category `13 Money` is better than
   separate folders for investments, budgets, savings, expenses. Fewer choices
   means less confusion.

3. **Nothing in area or category folders.** All files live in ID folders. If
   you find a file at the area or category level, it needs to move into an ID.

4. **"The goal is to reduce mental burden. It is not to use up all the
   numbers."** Don't create IDs just to fill gaps. Create them when content
   genuinely needs a new home.
