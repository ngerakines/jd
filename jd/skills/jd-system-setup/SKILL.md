---
name: jd-system-setup
description: >
  Initialize a new Johnny.Decimal system by creating the folder structure,
  standard zeros, and initial JDex. Use this skill when the user wants to
  set up a new JD system, create their JD folder structure, initialize a
  system, start organizing with Johnny.Decimal, or says things like "set up
  my JD system," "create a new system," "initialize JD," "build my folder
  structure," or "I want to start using Johnny.Decimal." Handles both
  single-system and multi-system (SYS.AC.ID) configurations.
---

# Johnny.Decimal System Setup

This skill creates a new Johnny.Decimal system from scratch — building the
folder structure, standard zeros, and initial JDex based on the user's needs.

---

## 1. Gather Requirements

Before creating anything, understand what the user needs.

### 1.1 Single or Multi-System?

Ask the user:

- **Single system**: One organizational domain (e.g., just personal life, or
  just work). No system prefix needed — folders use plain `AC.ID` notation.
- **Multi-system**: Multiple distinct domains (e.g., personal + work, or
  personal + work + child). Each system gets a `SYS` prefix using the
  `[A-Z][0-9][0-9]` format.

If the user already has JD systems, check the existing root to avoid
conflicts.

### 1.2 System Identity

For each system to create, gather:

- **System code** (multi-system only): A three-character `[A-Z][0-9][0-9]`
  identifier. Encourage memorable codes (e.g., `N42` for personal, `G24` for
  GitHub work). The code should be visually distinctive.
- **System name**: A plain-English name (e.g., "Personal", "GitHub", "Hannah").
- **Root location**: Where the system will live on disk. Common locations:
  - `~/Library/Mobile Documents/com~apple~CloudDocs/JD/` (iCloud Drive)
  - `~/Documents/JD/`
  - `~/JD/`

### 1.3 Areas and Categories

Walk the user through defining their areas. Provide guidance:

- Maximum 10 areas (including `00-09 System`)
- Each area covers a major life/work domain
- Fewer areas is better — compression over granularity
- Area `00-09` is always created automatically for system management

For each area, ask about categories:

- Maximum 10 categories per area
- Categories are "where you work" — collections of similar things
- Category names should be short and clear

For each category, ask if they have specific IDs in mind, or if they'll create
IDs as content arrives.

---

## 2. Create the Folder Structure

### 2.1 System Root

Create the system root folder:

- **Single system**: `[Root]/` (e.g., `~/JD/`)
- **Multi-system**: `[Root]/SYS Name/` (e.g., `~/JD/N42 Personal/`)

### 2.2 Area Folders

Create area folders using the format `AC-range Description`:

```
00-09 System
10-19 [Area name]
20-29 [Area name]
...
```

### 2.3 Category Folders

Within each area, create category folders using the format `AC Description`:

```
00 System management
01 [Category name]
11 [Category name]
12 [Category name]
...
```

### 2.4 Standard Zeros (System Level)

Always create these in `00-09 System/00 System management/`:

| Folder | Purpose |
|--------|---------|
| `00.00 JDex.md` | Master index (file, not folder) |
| `00.01 Inbox/` | Unsorted capture bucket |
| `00.02 Tasks.md` | Action items and follow-ups (file) |
| `00.03 Processing log.md` | Record of inbox processing sessions (file) |

Optionally create if the user wants them:

| Folder | Purpose |
|--------|---------|
| `00.04 Needs review/` | Items awaiting user decision |
| `00.08 Someday/` | Non-urgent ideas and parking lot |
| `00.09 Archive/` | Completed or stale items |

### 2.5 ID Folders

If the user specified initial IDs during requirements gathering, create them
using the format `AC.ID Description`:

```
11.01 Mortgage & title/
11.02 Property tax/
11.03 Homeowners insurance/
```

---

## 3. Generate the JDex

Create `00.00 JDex.md` with a complete listing of every area, category, and
ID in the system.

### JDex Format

```markdown
# [SYS] JDex

## 00-09 System
- 00.00 JDex ← this file
- 00.01 Inbox
- 00.02 Tasks
- 00.03 Processing log

## 10-19 [Area name]

### 11 [Category name]
- 11.01 [ID description]
- 11.02 [ID description]

### 12 [Category name]
- 12.01 [ID description]

## 20-29 [Area name]
...
```

### JDex Rules

- Every folder that exists must have a JDex entry
- Every JDex entry must have a corresponding folder
- +SUB entries are listed inline under their parent ID
- Cross-system references use arrow notation: `→ N42.34.01`

---

## 4. Initialize Task and Log Files

### 4.1 Tasks File

Create `00.02 Tasks.md`:

```markdown
# [System name] Tasks

Tasks extracted during inbox processing and daily use.

---

<!-- Add tasks below this line -->
```

### 4.2 Processing Log

Create `00.03 Processing log.md`:

```markdown
# [System name] Processing Log

Record of inbox processing sessions.

---

<!-- Processing entries will be appended below -->
```

---

## 5. Verification

After creating the structure:

1. **List the complete folder tree** to depth 3 and present it to the user
   for review.
2. **Display the JDex** so the user can verify all entries are correct.
3. **Confirm iCloud sync** (if applicable): Verify folders appear correctly.
4. **Note next steps**: Suggest the user start capturing items into their
   inbox and run `/jd:process-inbox` when ready.

---

## 6. Multi-System Considerations

When setting up multiple systems at once:

- Each system gets its own independent JDex
- Each system gets its own standard zeros
- Cross-system references should be noted during setup if known
  (e.g., "V06.32.01 will reference N42.34.01 for health insurance")
- Suggest the user set up one system fully before moving to the next

---

## 7. Templates for Common System Types

Offer these as starting points when the user isn't sure what areas to create:

### Personal Life System

```
00-09 System
10-19 Home & property
20-29 Family & relationships
30-39 Money & legal
40-49 Health & wellness
50-59 Projects & hobbies
60-69 Travel & experiences
```

### Work System

```
00-09 System & employment
10-19 [Primary work domain]
20-29 Career development
30-39 Professional visibility
```

### Child/Dependent System

```
00-09 System
10-19 Identity & documents
20-29 School / academics
30-39 Health & medical
40-49 Activities & interests
```

These are suggestions, not prescriptions. The user's actual needs should
drive the structure.
