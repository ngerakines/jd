# Johnny.Decimal Plugin

Manage [Johnny.Decimal](https://johnnydecimal.com/) systems with AI-assisted inbox processing, classification, JDex maintenance, and system setup. Supports single and multi-system (`SYS.AC.ID`) configurations.

## Installation

```
claude plugins add jd-plugins/jd
```

## Commands

| Command | Description |
|---------|-------------|
| `/jd:process-inbox [system-code]` | Process items in a JD inbox — classify, file, extract tasks, log actions |
| `/jd:jdex-audit [system-code]` | Compare JDex against folder structure, flag mismatches |
| `/jd:setup-system` | Initialize a new JD system with folders, standard zeros, and JDex |
| `/jd:sub-index [system-code] [category]` | Manage +SUB index files, assign next numbers, create entries |

## Skills

Skills activate automatically based on what you're doing:

| Skill | Triggers on |
|-------|------------|
| **jd-inbox-processor** | "process my inbox," "file my stuff," "sort my inbox," "triage my captures" |
| **jd-system-setup** | "set up my JD system," "create a new system," "initialize JD" |
| **jd-jdex-audit** | "audit my JDex," "check my system," "sync JDex," "verify my JD system" |
| **jd-sub-index** | "update sub index," "add a new project," "what's the next sub number" |

## Example Workflows

### Process your inbox after a busy week

```
> process my inbox

Discovered 2 systems with non-empty inboxes:
  P10 Personal: 8 items
  W20 Work: 3 items

Processing P10...
  ✓ 2026-02-05 W-2 Form.pdf → P10.33.02 (2025 tax return)
  ✓ 2026-02-06 Dentist receipt.pdf → P10.41.02 (Dental)
  ? Meeting notes.md — medium confidence: P10.24.04 (Alex notes) or W20.14.01 (1:1 with Morgan)?
  ...
```

### Set up a new system for a child

```
> /jd:setup-system

What kind of system? Child/dependent
System code: K30
System name: Jamie
Root: ~/Library/Mobile Documents/com~apple~CloudDocs/JD/

Creating K30 Jamie...
  00-09 System ✓
  10-19 Identity & documents ✓
  20-29 School ✓
  ...
```

### Audit after manual reorganization

```
> /jd:jdex-audit P10

JDex Audit: P10 Personal
  JDex entries: 87
  Filesystem folders: 89
  ✗ 2 undocumented folders found
  ✗ 1 name mismatch

Add 11.08 Garage storage to JDex? [y/n]
```

## Configuration

The plugin discovers your JD root automatically by checking:

1. `~/Library/Mobile Documents/com~apple~CloudDocs/JD/` (iCloud Drive)
2. `~/Documents/JD/`
3. `~/JD/`

If your JD root is elsewhere, the plugin will ask when you first use it.

### Multi-system support

The plugin handles both single-system and multi-system setups. Systems are identified by their `[A-Z][0-9][0-9]` prefix (e.g., `P10 Personal`, `W20 Work`).

## What is Johnny.Decimal?

Johnny.Decimal is a constraint-based organizational system with exactly three levels:

```
Area (10-19, 20-29, ..., 90-99)
  └── Category (11, 12, ..., 19)
        └── ID (11.01, 11.02, ..., 11.99)
```

Every file gets a unique numeric address. The system caps each level at ~10 items, making classification a bounded decision at every step. The **JDex** is the master index — the single source of truth for every ID in a system.

Learn more at [johnnydecimal.com](https://johnnydecimal.com/).

## License

Apache License 2.0. See [LICENSE](LICENSE).
