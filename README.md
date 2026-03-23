# Claude Plugins Marketplace

A personal marketplace for Claude Code plugins. Install plugins directly using Claude Code's built-in plugin system.

## Installation

Register this marketplace in Claude Code, then install any plugin:

```
/install <plugin-name>@claude-plugins-marketplace
```

Or browse available plugins with `/plugin > Discover`.

## Structure

- **`/plugins`** - Plugins developed and maintained by me
- **`/external_plugins`** - Third-party plugin references

## Plugin Structure

Each plugin follows the standard Claude Code plugin structure:

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json      # Plugin metadata (required)
├── .mcp.json            # MCP server configuration (optional)
├── commands/            # Slash commands (optional)
├── agents/              # Agent definitions (optional)
├── skills/              # Skill definitions (optional)
├── hooks/
│   └── hooks.json       # Lifecycle hooks (optional)
└── README.md            # Documentation
```

## Adding a Plugin

1. Create a new directory under `plugins/` with your plugin name
2. Add `.claude-plugin/plugin.json` with at minimum `name` and `description`
3. Add your skills, commands, hooks, MCP servers, and agents
4. Commit and push

## License

See each plugin directory for its own license.
