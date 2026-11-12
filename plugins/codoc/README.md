# Codoc plugin for Claude

The Codoc MCP server, the skill that teaches agents its review workflow, and skills for writing PRDs and design documents.

```
.claude-plugin/plugin.json   name, version
.mcp.json                    the MCP server: https://codoc.sassysaas.cc/mcp
icon.png                     the Codoc logo, for a directory listing (plugin.json has no icon key)
skills/codoc/SKILL.md        the review workflow, read on every use
skills/codoc/format.md       reference: agent Markdown, numbering attributes, results, failure codes
skills/codoc/examples/       a worked round with a partial failure; a review by two hats
skills/design-doc/SKILL.md   writing a design document a person will actually read
skills/prd/SKILL.md          writing a PRD, with mocks, before the design of important functionality
```

Voice comments (dictating a comment with the microphone) are a web-app feature; agents work with the text that is posted.

## Install in Claude Code

```
/plugin marketplace add chief-saassy/codoc-plugin
/plugin install codoc --marketplace chief-saassy/codoc-plugin
```

Or try it from a checkout of codoc-plugin: `claude --plugin-dir plugins/codoc`.

Then sign in: run `/mcp`, choose `codoc`, and choose Authenticate. Your browser opens Codoc's sign-in; it uses your Codoc account, and the plugin holds no secrets. Until then the Codoc tools are not available.

## Install on claude.ai and the desktop app

After adding the plugin, open its Connectors tab, add the Codoc connector, and press Connect to sign in with your Codoc account. An organisation admin may need to allow it.

Without the plugin, add the two parts yourself:

1. **Connector:** in Settings, Connectors, add a custom connector with the URL `https://codoc.sassysaas.cc/mcp`, then press Connect.
2. **Skills:** zip `skills/codoc/` and upload it under Settings, Capabilities, Skills. Do the same with `skills/design-doc/` and `skills/prd/` if you want them.

## Install in ChatGPT

Download [`chatgpt/codoc-chatgpt.zip`](https://github.com/chief-saassy/codoc-plugin/raw/main/chatgpt/codoc-chatgpt.zip) from codoc-plugin and upload it as a plugin in ChatGPT. It carries the Codoc MCP server and every skill here; when ChatGPT first uses Codoc it asks you to sign in with your Codoc account.

The ZIP is built from this folder by `scripts/chatgpt-plugin.mjs` in the main Codoc repository, after each deploy. Its manifest and MCP config follow ChatGPT's format (`plugin.json`, `mcp.json`, and `.codex-plugin/` for older clients), so don't edit them by hand.

## Update

An agent tells you when a newer plugin is out. You can keep working until you update, unless the server refuses the agent's call with `outdated_plugin`: meanwhile the agent fetches the latest skills from the Codoc server (`latest_skill`) and follows those. Updating is still better, since then it has them from the start.

- **Claude Code:** run `/plugin`, update Codoc, then run `/reload-plugins`, or start a new session.
- **claude.ai and the desktop app, with the plugin:** from the Add menu, open Manage marketplaces and check that it has synced the latest commit. Then update the plugin.
- **ChatGPT:** download `chatgpt/codoc-chatgpt.zip` again and upload it over the old version.
- **claude.ai and the desktop app, with an uploaded skill:** zip `skills/codoc/` from this version and upload it again under Settings, Capabilities, Skills. Do the same with `skills/design-doc/` and `skills/prd/` if you use them.

## Self-hosting

The plugin points at the hosted Codoc. For your own deployment, fork codoc-plugin and change the one `url` in `.mcp.json` to `https://<your domain>/mcp` (or `https://<your domain>/<base path>/mcp`), or add your endpoint as your own connector instead.

## Versioning

The `codoc` skill is versioned with the server. A change to the projection format or a tool signature bumps the minor `version` in `plugin.json`, along with the version and `format_version` that `skills/codoc/SKILL.md` names, which must match the server's. Agents told that the server expects a newer version stop and ask for an update. Anything else, such as a new skill, is a patch: installed plugins only receive a new version, and the agents on the last one keep working. Every bump also moves the version in the description and body of `SKILL.md` and the server's `LATEST_PLUGIN`, because every tool answer tells agents the latest version, and agents behind it encourage the user to update and carry on.

The version moves once per release, not once per change. A pull request that changes this folder leaves the version alone, unless it changes the format or a tool signature (the minor bump above, which the server needs at once). The deploy workflow moves the patch itself (`scripts/bump-plugin.mjs`), when this folder changed since the last deploy and nothing has moved it yet, and commits it to main as "Plugin x.y.z, for this release". The plugin is published here after each successful deploy, from the commit that went out, so the version people install is always the one the server runs with.

## Testing

The skill is tested against the real Codoc MCP tools in the main Codoc repository.
