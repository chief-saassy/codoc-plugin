# Codoc plugin for Claude

The Codoc MCP server plus the skill that teaches agents its review workflow.

```
.claude-plugin/plugin.json   name, version, icon
.mcp.json                    the MCP server: https://codoc.sassysaas.cc/mcp
icon.png                     the Codoc logo
skills/codoc/SKILL.md        the review workflow, read on every use
skills/codoc/format.md       reference: agent Markdown, numbering attributes, results, failure codes
skills/codoc/examples/       a worked round with a partial failure; a review by two hats
```

## Install in Claude Code

```
/plugin marketplace add chief-saassy/codoc-plugin
/plugin install codoc --marketplace chief-saassy/codoc-plugin
```

Or try it from a checkout of codoc-plugin: `claude --plugin-dir plugins/codoc`.

The first tool call signs in through your browser. Codoc's OAuth uses your Codoc account, and the plugin holds no secrets.

## Install on claude.ai and the desktop app

1. **Connector:** in Settings, Connectors, add a custom connector with the URL `https://codoc.sassysaas.cc/mcp`, then press Connect to sign in. An organisation admin may need to add it.
2. **Skill:** zip `skills/codoc/` and upload it under Settings, Capabilities, Skills.

## Self-hosting

The plugin points at the hosted Codoc. For your own deployment, fork codoc-plugin and change the one `url` in `.mcp.json` to `https://<your domain>/mcp` (or `https://<your domain>/<base path>/mcp`), or add your endpoint as your own connector instead.

## Versioning

The skill is versioned with the server. A change to the projection format or a tool signature bumps `version` in `plugin.json`, along with the `format_version` that `SKILL.md` names, which must match the server's agent format version.

## Testing

The skill is tested against the real Codoc MCP tools in the main Codoc repository.
