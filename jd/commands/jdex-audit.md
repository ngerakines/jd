---
description: Audit a Johnny.Decimal system by comparing the JDex against the folder structure
argument-hint: "[system-code]"
allowed-tools: [Read, Glob, Grep, Bash, Write]
---

# Audit JDex

Compare a system's JDex against its actual folder structure, flagging
orphaned entries, undocumented folders, name mismatches, and +SUB index
issues.

## Arguments

`$ARGUMENTS` may contain a system code (e.g., `N42`) to audit a specific
system. If no argument is provided, list all available systems and ask which
to audit.

## Workflow

1. **Load** the target system's JDex.
2. **Snapshot** the filesystem to depth 3.
3. **Compare** JDex entries against folders:
   - Orphaned JDex entries (in JDex, no folder)
   - Undocumented folders (folder exists, not in JDex)
   - Name mismatches between JDex and folder names
4. **Check** +SUB index files against +SUB folders.
5. **Report** findings in a structured table.
6. **Offer fixes**: Add missing JDex entries, update +SUB indexes, or flag
   items that need user decisions.

## Safe Fixes

- Adding undocumented folders to the JDex
- Updating +SUB index files with missing entries

## Requires User Decision

- Orphaned JDex entries (create folder or remove entry?)
- Name mismatches (which name is correct?)
- Deleting or renaming any folder

## Examples

```
/jd:jdex-audit
/jd:jdex-audit N42
/jd:jdex-audit V06
```
