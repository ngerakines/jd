---
description: Manage +SUB index files — scan entries, assign next numbers, and create new +SUB entries
argument-hint: "[system-code] [category]"
allowed-tools: [Read, Glob, Grep, Bash, Write]
---

# Manage +SUB Index

Scan a +SUB category, update its index file, and optionally create new
+SUB entries with the next sequential number.

## Arguments

`$ARGUMENTS` may contain:

- A system code (e.g., `N42`) and category number (e.g., `51`) to target
  a specific +SUB category.
- Just a system code to list all +SUB categories in that system.
- No arguments to discover all +SUB categories across all systems.

## Workflow

1. **Locate** the target +SUB category and its index file.
2. **Scan** existing +SUB folders (matching `AC.ID+NNNN` or `AC.ID+CODE`).
3. **Compare** folder entries against the index file.
4. **Update** the index with any missing entries.
5. **Report** the current state: total entries, active/archived counts,
   next available number.

## Creating New Entries

When the user wants to add a new +SUB entry:

1. Determine the next identifier (sequential number or user-provided code).
2. Create the +SUB folder.
3. Update the index file.
4. Update the JDex.

## Examples

```
/jd:sub-index
/jd:sub-index N42
/jd:sub-index N42 51
```
