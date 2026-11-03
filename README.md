# Codoc plugin for Claude

Review, comment on and suggest changes to [Codoc](https://codoc.sassysaas.cc) documents from Claude. The plugin adds the Codoc MCP server and a skill that teaches Claude Codoc's review workflow.

## Install in Claude Code

```
/plugin marketplace add chief-saassy/codoc-plugin
/plugin install codoc@codoc
```

When the plugin is enabled, Claude Code asks for your Codoc MCP URL: `https://<your Codoc domain>/mcp` (for the hosted Codoc, `https://codoc.sassysaas.cc/mcp`). The first tool call signs you in through your browser with your Codoc account; the plugin holds no secrets.

## Install on claude.ai and the desktop app

1. **Connector:** add a custom connector with the same MCP URL.
2. **Skill:** zip `plugins/codoc/skills/codoc/` and upload it under Settings → Capabilities → Skills.

## Layout

```
.claude-plugin/marketplace.json     the marketplace, listing the plugin
plugins/codoc/                      the plugin (see its README)
```
