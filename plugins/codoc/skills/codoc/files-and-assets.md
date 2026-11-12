# Sending files, and assets

## Sending a file

**Send files to a link, not in the tool call.** Whenever you can run commands, use the link for every file: an image, a chart's data, an asset, a Markdown file. The file goes straight from disk to Codoc, so its contents never pass through you, nothing is lost or garbled on the way, and it costs no context. Base64 in a tool call is the fallback, for when there is no shell or the link can't be reached.

1. `request_upload(sha256, size, target)`. Get the hash with `shasum -a 256 file` and the size in bytes with `wc -c < file`. `target` says what it's for: `{for: "image", doc}`, `{for: "data", doc}`, `{for: "asset", title, description, type}`, `{for: "asset_version", asset}`, or `{for: "document", title, request_key}`.
2. Run the command in `how`. Its answer is what the tool call would have given you.

The link works once, for that one file, for 15 minutes. Sending the same file again is safe. The same checks apply as in a tool call.

**If the link can't be reached.** Some places you run in, such as Claude's code sandbox on claude.ai, block addresses they don't know: the command fails with an error from a proxy, or "host not allowed", rather than an answer from Codoc. Then send this file in the tool call (`upload_image`, `upload_data`, `create_asset`, `update_asset` or `create_document`), and tell the user once how to let links through, naming the address in `how` (its host, such as `codoc.sassysaas.cc`):

- **claude.ai on a Team or Enterprise plan:** an organisation owner opens Organization settings, then Capabilities, then Code execution, chooses network egress to package managers and specific domains, and adds the host under the additional allowed domains.
- **claude.ai on another plan:** look in Settings, then Capabilities, for the code execution network setting. If it offers no list of allowed domains, links can't be used there yet, and files go in the tool call.
- **Claude Code:** a sandboxed shell takes the host in `sandbox.network.allowedDomains` in `settings.json`, and a cloud environment in its own custom network allowlist.

Don't try to get around a block another way.

## Assets

Assets are what an organisation shares across documents: chart themes, Mermaid themes and images such as a logo. A theme is for charts or for Mermaid, never both. A chart's data is never an asset: it stays with its chart, and if it is needed again, add it again.

- **Look first.** `list_assets(type?)` shows what you can see. Use what exists rather than making your own.
- **Show one in chat.** `view_asset(asset, version?)` gives an image as an image beside the answer, so show it to the user, and a theme as its JSON.
- **Use an image.** `use_asset(asset, doc)` answers its `markdown`: add that with an edit.
- **An image starts private,** since it may show something sensitive. Pass `orgAccess: "viewer"` only for one meant to be shared, such as a logo, and ask the user if you aren't sure. Themes start visible to the organisation.
- **Add one when asked.** `create_asset(title, description, type, data)`, with a description saying what it's for. `type` is `chart_theme`, `mermaid_theme` or `image`. Only agents create themes: people can upload images and make small tweaks, so anything more detailed in a theme is your job.
- **Change one.** `update_asset(asset, ...)` with new `data` adds a version and makes it live. `restore_asset(asset, version)` makes an earlier version live. Neither changes any document: documents keep what they pinned.
