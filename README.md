# Johnny.Decimal Plugins

Claude Cowork plugins for the [Johnny.Decimal](https://johnnydecimal.com/) organizational system.

## Plugins

| Plugin | Description |
|--------|-------------|
| [jd](jd/) | Manage Johnny.Decimal systems with inbox processing, JDex maintenance, system setup, and +SUB index management |

## Installation

Install a plugin by name:

```
claude plugins add jd-plugins/jd
```

## What is Johnny.Decimal?

Johnny.Decimal is a constraint-based organizational system that assigns every file a unique numeric address using exactly three levels: Areas, Categories, and IDs. The `AC.ID` notation (e.g., `43.02`) is both human-memorable and machine-parseable, making it well-suited for AI-assisted workflows.

Learn more at [johnnydecimal.com](https://johnnydecimal.com/).

## Contributing

Each plugin is a self-contained directory with:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── commands/                 # Slash commands
│   └── command-name.md
├── skills/                   # Domain knowledge (auto-triggered)
│   └── skill-name/
│       └── SKILL.md
├── README.md
└── LICENSE
```

To add a new plugin, create a directory following this structure and add an entry to `.claude-plugin/marketplace.json`.

## License

Apache License 2.0. See [LICENSE](LICENSE).
