---
description: Initialize a new Johnny.Decimal system with folder structure, standard zeros, and JDex
argument-hint: ""
allowed-tools: [Read, Glob, Grep, Bash, Write]
---

# Set Up a Johnny.Decimal System

Create a new Johnny.Decimal system from scratch — folders, standard zeros,
and initial JDex.

## Arguments

No required arguments. The command will walk the user through an interactive
setup process.

## Workflow

1. **Ask** whether this is a single-system or multi-system setup.
2. **Gather** system identity: code (if multi-system), name, and root location.
3. **Define areas**: Walk through area definitions (max 10 including 00-09).
4. **Define categories**: For each area, define categories (max 10 per area).
5. **Define initial IDs**: Optionally pre-create ID folders within categories.
6. **Create structure**: Build all folders on disk.
7. **Generate JDex**: Write the complete index at `00.00 JDex.md`.
8. **Initialize files**: Create `00.02 Tasks.md` and `00.03 Processing log.md`.
9. **Verify**: Display the complete folder tree for user review.

## Templates

Offer starting templates for common system types:

- **Personal life**: Home, family, money, health, projects, travel, hobbies
- **Work**: System/employment, primary work domain, career, visibility
- **Child/dependent**: Identity, school, health, activities

## Examples

```
/jd:setup-system
```
