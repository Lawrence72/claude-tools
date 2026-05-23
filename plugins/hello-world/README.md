# hello-world

A simple demonstration plugin for the `claude-tools` marketplace. When invoked, Claude Code responds with exactly `Hello, World!` — nothing more, nothing less.

## Installation

First, add the `claude-tools` marketplace to Claude Code (if you haven't already):

```
/plugins add marketplace github:Lawrence72/claude-tools
```

Then install this plugin:

```
/plugins install hello-world
```

## Usage

```
/hello-world
```

Claude will respond with:

> Hello, World!

## Purpose

This plugin serves as a minimal working example of the `claude-tools` marketplace plugin structure. It demonstrates:

- The required `.claude-plugin/plugin.json` manifest
- A user-invoked skill using `skills/<name>/SKILL.md`
- How a skill's markdown body becomes the instruction set for Claude

Use this as a reference when building your own plugins.
