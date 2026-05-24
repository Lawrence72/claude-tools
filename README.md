# claude-tools

A Claude Code plugin marketplace with a growing collection of tools and skills.

## Add This Marketplace to Claude Code

```
/plugins add marketplace github:Lawrence72/claude-tools
```

## Available Plugins

| Plugin | Category | Description |
|--------|----------|-------------|
| [feature-planner](./plugins/feature-planner) | planning | Guided feature planning — interviews the user, explores the codebase, and writes a structured implementation plan to `~/.claude/plans/` |
| [ticket-estimator](./plugins/ticket-estimator) | planning | Takes a PM ticket, builds a full implementation plan with real file paths and code skeletons, then derives a velocity cost estimate from that plan via `/estimate` |

## Install a Plugin

After adding this marketplace, install any plugin with:

```
/plugins install <plugin-name>
```

For example:

```
/plugins install ticket-estimator
```

## Contributing a Plugin

Each plugin lives in its own subdirectory under `plugins/` and must follow this structure:

```
plugins/
└── your-plugin-name/
    ├── .claude-plugin/
    │   └── plugin.json          # Required — plugin manifest
    ├── skills/
    │   └── your-skill/
    │       └── SKILL.md         # User or model-invoked skill
    └── README.md                # Plugin documentation
```

### `plugin.json` format

```json
{
  "name": "your-plugin-name",
  "description": "What your plugin does",
  "author": {
    "name": "Your Name"
  }
}
```

### `SKILL.md` format

```markdown
---
name: your-skill-name
description: Short description of when/how to invoke this skill
---

Instructions for Claude go here. This markdown body is exactly what
Claude receives as its instruction set when the skill is invoked.
```

Once your plugin is ready, add an entry to `.claude-plugin/marketplace.json` and open a pull request.

## Marketplace Structure

```
claude-tools/
├── .claude-plugin/
│   └── marketplace.json    # Central plugin catalog
├── plugins/
│   └── hello-world/        # One folder per plugin
└── README.md
```
