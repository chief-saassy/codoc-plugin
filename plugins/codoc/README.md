# Codoc plugin for Claude

The Codoc MCP server plus the skill that teaches agents its review workflow.

```
.claude-plugin/plugin.json   name, version, and the server URL setting
.mcp.json                    the MCP server; the URL comes from that setting
skills/codoc/SKILL.md        the review workflow, read on every use
skills/codoc/format.md       reference: agent Markdown, numbering attributes, results, failure codes
skills/codoc/examples/       a worked round with a partial failure; a review by two hats
```

## The server URL

Codoc's domain is set per deployment, so the plugin doesn't name one; the hosted Codoc is `https://codoc.sassysaas.cc/mcp`. The endpoint is `https://<domain>/mcp`, or `https://<domain>/<BASE>/mcp` under a base path.

## Install in Claude Code

Add the plugin from its marketplace, or try it from a checkout:

```sh
claude --plugin-dir plugins/codoc
```

When the plugin is enabled, Claude Code asks for the Codoc MCP URL. To change it later, use `/config`. In managed or scripted set-ups, set it in `settings.json`:

```json
{ "pluginConfigs": { "codoc@<marketplace>": { "options": { "server_url": "https://docs.example.com/mcp" } } } }
```

The first tool call signs in through your browser. Codoc's OAuth uses your Codoc account, and the plugin holds no secrets.

## Install on claude.ai and the desktop app

Plugins aren't the unit of installation there, so add the two parts separately:

1. **Connector:** add a custom connector with the same MCP URL. An organisation admin may need to do this.
2. **Skill:** zip `skills/codoc/` and upload it under Settings, Capabilities, Skills.

## Versioning

The skill is versioned with the server. A change to the projection format or a tool signature bumps `version` in `plugin.json`, along with the `format_version` that `SKILL.md` names, which must match the server's agent format version.

## Testing

The skill is tested against the real Codoc MCP tools in the main Codoc repository.
