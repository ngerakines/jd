---
name: jd-jdex-audit
description: >
  Audit a Johnny.Decimal system by comparing the JDex against the actual
  folder structure, flagging mismatches, orphaned entries, and undocumented
  folders. Use this skill when the user wants to verify their JD system,
  check for inconsistencies, sync their JDex with the filesystem, or says
  things like "audit my JDex," "check my system," "sync JDex," "verify my
  JD system," "is my JDex up to date," or "find orphaned folders."
---

# Johnny.Decimal JDex Audit

This skill compares a system's JDex (the authoritative index) against the
actual folder structure on disk, identifying mismatches and suggesting fixes.

---

## 1. Locate the System

### 1.1 Find the JD Root

Check common locations:

- `~/Library/Mobile Documents/com~apple~CloudDocs/JD/` (iCloud Drive)
- `~/Documents/JD/`
- `~/JD/`

If the user specifies a system code (e.g., "audit my N42 system"), target
that specific system. Otherwise, list all available systems and ask which
to audit.

### 1.2 Load the JDex

Read the JDex file at:

```
SYS/00-09 */00 */00.00 *JDex*
```

Parse it into a structured list of areas, categories, and IDs. Each entry
should capture:

- The AC.ID (or SYS.AC.ID) address
- The description/name
- Any +SUB entries

### 1.3 Snapshot the Filesystem

List all folders to depth 3 under the system root. Parse folder names to
extract AC.ID addresses, area ranges, and category numbers.

---

## 2. Compare JDex vs. Filesystem

Run three checks:

### 2.1 Orphaned JDex Entries

JDex entries that have no corresponding folder on disk.

These indicate either:
- A folder was deleted without updating the JDex
- A planned ID that was never created
- A typo in the JDex entry

### 2.2 Undocumented Folders

Folders on disk that have no corresponding JDex entry.

These indicate either:
- A folder was created without updating the JDex
- The JDex fell out of sync during manual organization
- A folder created by another tool or process

### 2.3 Name Mismatches

JDex entries whose description doesn't match the folder name, or folders
whose name doesn't follow the expected `AC.ID Description` format.

---

## 3. Check +SUB Indexes

For categories that use +SUB extensions:

### 3.1 Locate Index Files

+SUB categories typically have an index file (e.g., `51.01 ATProto project
index.md`). Find these by looking for files matching `*.md` at the category
or ID level that contain +SUB listings.

### 3.2 Compare Index vs. Folders

- List all `+NNNN` or `+CODE` folders within the ID
- Compare against the index file entries
- Flag missing or extra entries

---

## 4. Report Findings

Present a clear audit report to the user:

### Report Format

```markdown
## JDex Audit: [System name]

**Audit date:** YYYY-MM-DD
**JDex location:** [path]
**System root:** [path]

### Summary

| Check | Count |
|-------|-------|
| JDex entries | N |
| Filesystem folders | N |
| Orphaned JDex entries | N |
| Undocumented folders | N |
| Name mismatches | N |
| +SUB index issues | N |

### Orphaned JDex Entries (in JDex but no folder)

| AC.ID | Description | Suggested Action |
|-------|-------------|-----------------|
| 11.05 | Home improvement | Create folder or remove from JDex |

### Undocumented Folders (folder exists but not in JDex)

| Folder | Path | Suggested Action |
|--------|------|-----------------|
| 11.08 Garage storage | 10-19/11/11.08 | Add to JDex |

### Name Mismatches

| AC.ID | JDex says | Folder says | Suggested Action |
|-------|-----------|-------------|-----------------|
| 11.03 | Home insurance | Homeowners insurance | Reconcile names |

### +SUB Index Issues

| Category | Issue | Details |
|----------|-------|---------|
| 51.01 | Missing from index | +0004 folder exists, not in index |
```

---

## 5. Offer Fixes

After presenting the report, offer to fix issues:

### Safe Fixes (do automatically with confirmation)

- **Add undocumented folders to JDex**: Append new entries matching the
  folder names.
- **Update +SUB index files**: Add missing entries to index files.

### Requires User Decision

- **Orphaned JDex entries**: Ask whether to create the missing folder or
  remove the JDex entry.
- **Name mismatches**: Ask which name is correct (JDex or folder) and
  update the other.

### Never Do Automatically

- **Delete folders**: Even if they appear orphaned, never delete without
  explicit user confirmation.
- **Rename folders**: Name changes can break references. Always confirm first.
- **Create new areas or categories**: These are structural decisions.

---

## 6. Regenerate JDex (Optional)

If the user requests it, offer to regenerate the JDex entirely from the
folder structure:

1. Walk the filesystem and build a complete list of areas, categories, and IDs.
2. Preserve any JDex-only metadata (cross-system references, notes, +SUB
   descriptions) from the existing JDex.
3. Write the new JDex, merging filesystem reality with JDex metadata.
4. Present the result for user approval before overwriting.

**Warning**: Regeneration loses any JDex entries that don't have corresponding
folders. Always confirm before proceeding.

---

## 7. Multi-System Audit

When auditing multiple systems:

1. Audit each system independently.
2. Present per-system reports.
3. Optionally check cross-system references: if N42.34.01 is referenced by
   V06.32.01, verify that both the source and the reference exist.
4. Provide a combined summary at the end.
