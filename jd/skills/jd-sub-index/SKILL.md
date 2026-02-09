---
name: jd-sub-index
description: >
  Manage +SUB index files in Johnny.Decimal categories that use the
  AC.ID+NNNN or AC.ID+CODE extension pattern. Use this skill when the user
  wants to update a +SUB index, add a new project or entry to a +SUB
  category, check what the next +SUB number is, or manage +SUB entries.
  Trigger on phrases like "update sub index," "add a new project," "what's
  the next sub number," "manage +SUB entries," "new +SUB entry," "add to
  my project list," or "update my project index."
---

# Johnny.Decimal +SUB Index Manager

This skill maintains +SUB index files within categories that use the
`AC.ID+NNNN` or `AC.ID+CODE` extension pattern. It keeps index files in
sync with the actual +SUB folders and helps create new entries.

---

## 1. Understand +SUB Extensions

+SUB extends the `AC.ID` notation for items that repeat across many instances
within a single ID.

### Numeric +SUB: `AC.ID+NNNN`

Zero-padded four-digit sequential number:

```
53.01+0001  weather-api
53.01+0002  task-runner
53.01+0003  auth-dashboard
```

### Named +SUB: `AC.ID+CODE`

Short alphanumeric code:

```
11.01+REDD  The Red Door
11.01+OAKS  Oak Street Cafe
11.01+RIVR  Riverside Grill
```

### When Each Format is Used

| Numeric +NNNN | Named +CODE |
|---|---|
| Items are primarily sequential | Items have natural short codes |
| No inherent abbreviation | Abbreviations are obvious and useful |
| Expect many entries (20+) | Expect fewer, memorable entries |
| Example: projects, trips, campaigns | Example: accounts, venues, people |

---

## 2. Locate +SUB Categories

### 2.1 Find the System

Check common JD root locations and identify the target system. If the user
specifies a system code and category, go directly there. Otherwise, scan
for +SUB categories.

### 2.2 Identify +SUB Categories

A category uses +SUB if it contains folders matching the pattern:

```
AC.ID+NNNN Description/
AC.ID+CODE Description/
```

Scan the system's categories and list those with +SUB folders.

### 2.3 Find the Index File

Each +SUB category should have an index file. Common patterns:

- `AC.ID [Category] index.md` (e.g., `51.01 Open-source project index.md`)
- `AC.00 [Category] index.md` (e.g., `51.00 Projects area index.md`)

If no index file exists, offer to create one.

---

## 3. Scan Existing Entries

List all +SUB folders within the target category/ID. For each entry, capture:

- The +SUB identifier (number or code)
- The description (from folder name)
- Whether the folder contains content or is empty
- Creation date (from folder metadata or earliest file date)

---

## 4. Update the Index

### 4.1 Index File Format

The index file should list every +SUB entry with its identifier, name,
creation date, and status:

```markdown
# [Category name] Index

| +SUB | Name | Created | Status |
|------|------|---------|--------|
| +0001 | weather-api | 2025-06-01 | active |
| +0002 | task-runner | 2025-08-15 | active |
| +0003 | auth-dashboard | 2025-11-01 | archived |
```

For named +SUB:

```markdown
# [Category name] Index

| +SUB | Name | Created | Status |
|------|------|---------|--------|
| +REDD | The Red Door | 2025-01-10 | active |
| +OAKS | Oak Street Cafe | 2025-03-22 | active |
| +RIVR | Riverside Grill | 2025-05-01 | active |
```

### 4.2 Sync Process

1. Read the existing index file (if it exists).
2. Compare index entries against actual +SUB folders.
3. Add any folders that are missing from the index.
4. Flag any index entries whose folders don't exist.
5. Update the index file with the reconciled list.

### 4.3 Preserve Existing Format

If the user's index file uses a different format than the template above,
match their existing format. Don't impose a new structure on an established
index.

---

## 5. Create New +SUB Entries

When the user wants to add a new entry:

### 5.1 Determine the Next Identifier

**For numeric +SUB:**

1. List all existing +NNNN entries in the category.
2. Find the highest number.
3. The next entry is that number + 1, zero-padded to 4 digits.
4. Example: highest is `+0003` → next is `+0004`.

**For named +SUB:**

1. Ask the user for the entry name.
2. Derive a short code (typically 4 uppercase letters from the name).
3. Verify the code doesn't conflict with existing entries.
4. Example: "Midnight Sun Brewing" → `+MSUN`.

### 5.2 Create the Folder

Create the +SUB folder:

```
AC.ID+NNNN Description/
```

or:

```
AC.ID+CODE Description/
```

### 5.3 Update the Index

Append the new entry to the index file.

### 5.4 Update the JDex

Add the new +SUB entry to the system's JDex under its parent ID:

```markdown
- 53.01 Open-source projects
  - 53.01+0001 weather-api
  - 53.01+0002 task-runner
  - 53.01+0003 auth-dashboard
  - 53.01+0004 [new entry]     ← added
```

---

## 6. Archiving +SUB Entries

When a +SUB entry is no longer active:

1. Update the status in the index file from "active" to "archived".
2. Do **not** move or delete the folder — archived entries stay in place.
3. Optionally note the archive date in the index.
4. The folder and its contents remain accessible for reference.

---

## 7. Reporting

After any update, present a summary:

```
+SUB Index Update: [Category name]

Entries: N total (N active, N archived)
Added: [list of new entries, or "none"]
Issues: [list of mismatches, or "none"]

Next available number: +NNNN
```
