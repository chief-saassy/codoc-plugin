# Codoc plugin for Claude

Review, comment on and suggest changes to [Codoc](https://codoc.sassysaas.cc) documents from Claude. The plugin adds the Codoc MCP server and a skill that teaches Claude Codoc's review workflow.

## Install in Claude Code

```
/plugin marketplace add chief-saassy/codoc-plugin
/plugin install codoc@codoc
```

The plugin connects to the hosted Codoc at `https://codoc.sassysaas.cc/mcp`. The first tool call signs you in through your browser with your Codoc account; the plugin holds no secrets.

## Install on claude.ai and the desktop app

1. **Connector:** in Settings → Connectors, add a custom connector with the URL `https://codoc.sassysaas.cc/mcp`, then press **Connect** to sign in with your Codoc account. An organisation admin may need to add it.
2. **Skill:** zip `plugins/codoc/skills/codoc/` and upload it under Settings → Capabilities → Skills.

## Self-hosting

For your own Codoc deployment, fork this repository and change the `url` in `plugins/codoc/.mcp.json` to `https://<your domain>/mcp`, or add your endpoint as your own connector.

## Layout

```
.claude-plugin/marketplace.json     the marketplace, listing the plugin
plugins/codoc/                      the plugin (see its README), published from chief-saassy/codoc; edit it there
```
