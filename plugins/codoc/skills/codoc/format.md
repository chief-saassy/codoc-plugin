# Codoc format reference

Read this when `SKILL.md` doesn't settle a question about the text you were given.

## Agent Markdown

Standard Markdown (a strict subset of Pandoc's `commonmark_x`), one sentence per line, plus:

| you see | it is | you may |
|---|---|---|
| `## Fees {#fees}`, `{#late}` alone on a line, `- {#b}` | a block's name. A block without one shows none | use it as `block`, `window` and in `[](#fees){.live}` |
| `<!-- 1.1 -->`, `<!-- (b), cited as 1.1(b) -->` | a computed number, and for a list item how to cite it | quote it to people; it is ignored on write |
| `[](#fees){.live} <!-- section 1.1 -->` | a live reference: it shows the number, or a title or first words | write `[](#fees){.live}`; the number follows. Exports write "(see Rollout)" for a heading with no number |
| `[the fees](#fees)` | a link in its own words to a block here. It warns if the block goes | write it. `[](#fees)` is refused |
| `[new]{.insertion #__s13 author="sam/claude"}` | suggested insertion | copy as is |
| `[old]{.deletion #__s14 author="adrian"}` | suggested deletion | copy as is |
| `[old]{.deletion #__s12 author="adrian"}[new]{.insertion suggestion="__s12" author="adrian"}` | suggested replacement | copy as is |
| `[60]{.deletion #__s16 author="sam/claude" of="__s12"}` | a deletion of text `__s12` proposed | copy as is |
| `[]{.insertion #__s3 author="adrian"}`, `[]{.deletion …}` on its own line | the whole block below is proposed, or proposed for removal | copy as is |
| `[]{.comment-start #__c7}`, `[]{.comment-end #__c7}` | where thread `__c7` is anchored | copy as is; can't be moved |
| `[]{.comment #__c8}` on its own line | a thread on the whole block below | copy as is |
| `<!-- __c7 @adrian 2026-10-24: … -->` | one comment, read-only | ignored on write; use a `reply` item |
| `[words](https://…)`, `[words](codoc:abc#fees)` | a link; `codoc:{docId}` is another Codoc document, `codoc:{docId}#label` a section of it | keep it as it is; a bad address is refused with `invalid_link` |
| `` `RELEASE_VERSION` ``, ``` `` a ` b `` ``` | inline code: a file path, a name, a command | write it with backticks, as in any Markdown. Inside, nothing is formatting and backslashes are text, except `\|` in a table |
| `\*`, `\{`, `\$`, `` \` `` | a backslash escape: plain text, never markup | copy it in `old`; don't add one in your own `new` |

A span's `author` reads `user`, then `/client` for anything but the web editor, and `hat="security"` if it has one. Decided suggestions and resolved threads aren't shown; `thread(doc, thread)` has their history. In tool results ids are spelled `__c7` and `__s12`; in tool parameters `c7` and `__c7` both work. Each comment and suggestion also has a `url`, which is what you give a person.

Only this format is read. The older forms (`{#sec:x}`, `@sec:x`, `block: "sec:x"`, `thread: "c7"`, CriticMarkup) are refused with `outdated_plugin`.

## Changing text that already carries suggestions

Change someone else's proposed text and you make your own suggestion on top of theirs. Say `[30]{.deletion #__s12 author="adrian"}[60]{.insertion suggestion="__s12" author="adrian"}` should be 45:

```json
{"edit":{"block":"fees-due","old":"[60]","new":"[45]"}}
```

The result reads `[30]{.deletion #__s12 author="adrian"}[60]{.deletion #__s16 author="sam/claude" of="__s12"}[45]{.insertion #__s16 author="sam/claude"}`. Changing your own proposed text amends it. Deleting your own proposed text removes it.

One edit that changes someone's proposed text and plain text as well is still one suggestion: it depends on theirs as a whole, so rejecting theirs rejects all of it. To keep the plain change independent, send it as its own edit. One suggestion can show several marked spans with the same id.

## Whole-block replacement

`{"edit":{"block":"fees","text":"…"}}` replaces the block. Send it as you read it, with every span and marker intact, and with your changes as plain text. It is refused if anyone else changed the block since you read it. Deleting a paragraph may take its thread lines with it; those threads become orphaned.

## Images

An image is a line `![alt text](codoc-image:sha256-…)` with its caption on the lines after it, up to a blank line. To add one, call `upload_image(doc, data)` first (base64 of the file, or the text of an SVG with `encoding: "utf8"`), then put the `markdown` line it returns, followed by a caption, in an edit, with a blank line before and after. Uploading the same bytes again gives the same `src`. The caption is text like any other; changing the alt text or the picture is a direct edit, refused in suggest mode. A `src` that was never uploaded is refused.

## Code blocks

A fenced block with its language, ```` ```ts ````, is shown as code with line numbers. Keep each line to 100 characters or fewer: that fits the page's text column, and a longer line makes the reader scroll sideways inside the block. Wrap long lines where the language allows, break a long shell command with `\` continuations, and shorten long literals (a hash, a URL, a JSON value) or move them to their own line, rather than letting a line run past 100.

## Mermaid diagrams

A ```` ```mermaid ```` fence is drawn as a diagram. Codoc parses every diagram an edit, a suggestion or `create_document` adds or changes, and refuses one that doesn't parse (`invalid_diagram`), so the owner never opens a document to a parse error. Diagrams you don't touch are never held against you, even broken ones.

- **No `;` in message, note or label text.** Mermaid reads `;` as the end of a statement, even after `:` in `A->>B: text`. Write a comma, or `#59;` if you need the semicolon itself.
- **Quote labels with special characters:** `A["Read (cached); retry"]`, not `A[Read (cached); retry]`: parentheses, brackets, braces or `;` in an unquoted flowchart label break it.
- Start with the diagram type on its own line (`flowchart TD`, `sequenceDiagram`, `classDiagram`, `stateDiagram-v2`, `erDiagram`, `gantt`, `pie`, `mindmap`, `timeline`, …).
- **Title a diagram in the text, not inside it:** a short line just above or below the fence, such as a bold one-line `**Figure: Sign-in after the move**` or a sentence introducing it. No `title` front matter, `title` lines or title nodes in the diagram. A title in the text reads, searches and exports like the rest of the document, people can comment on it, and the diagram stays smaller.
- A refusal names the diagram, the line (counted in the fence, from 1) and Mermaid's message; for an edit they are also in `details: {block, line, message}`. Fix that line and send the edit again. The rest of the batch still applies.
- A diagram already in the document that doesn't parse (someone wrote it in the app, or it predates the check) is a warning, `{block, kind: "invalid-diagram", line, error, message}`, in `open_review` and `read_document`. Don't fix it unless it is yours or the user asks; tell them.

## Structure attributes

Attributes go at the end of a heading line, on a list item's first line, and alone on the line directly above any other block. A new block needs a name only if something will refer to it: lowercase, unique, descriptive, and never starting `__`.

| attribute | on | meaning |
|---|---|---|
| `#label` | any block | its name |
| `.numbered`, `.unnumbered` | heading | number this heading or not, whatever the document's default |
| `numbering="schedule"` | heading | starts a series: "Schedule 1 — Services" |
| `.empty` | untitled heading | the number stands alone; what follows starts below it |
| `start=3` | numbered heading | this heading counts from 3: it reads 3, the next at its level 4, and the levels inside it count as usual under it. `start=1` restarts the numbering here |
| `format="(a)"` | item | this list's look from here. `(a)`, `a.`, `A`, `(i)`, `i`, `I`, `1.`, `(1)`, `01`, `bullet` |
| `start=3` | item | the list starts at 3 |
| `.continue` | item | carry on from the previous list, instead of restarting |
| `depth=2` | paragraph | the list depth its text lines up with, where indentation can't say |
| `.empty` | alone, then a blank line | an empty paragraph: `{.empty}`, or `{#x .empty}` to name it |

An image's caption is the lines after it, up to a blank line, and its name goes on the line above.

In this example, "This includes…" continues (a), and "pay the Fees" starts a new list:

```markdown
# Obligations <!-- 3 --> {#obligations}

## <!-- 3.1 --> {#duty}
The Supplier owes these duties.

The Supplier shall:

1. {#skill-care format="(a)"}
   provide the Services with reasonable skill and care; <!-- (a), cited as 3.1(a) -->

   This includes keeping its staff trained.

   1. {format="(i)"}
      by annual courses; <!-- (i), cited as 3.1(a)(i) -->

1. comply with the Customer's policies. <!-- (b), cited as 3.1(b) -->

The Customer shall:

1. {#pay format="(a)"}
   pay the Fees; <!-- (a), cited as 3.1(a) -->
```

Writing `{#pay format="(a)" .continue}` would make that (c).

Comments carry no meaning. `<!-- empty -->` is not how to say a number stands alone, and `<!-- 7 -->` doesn't set a number.

## Numbering

A document on the defaults shows no block. Otherwise `read_document` opens with one, which you change with an ordinary edit in edit mode (in suggest mode it is refused; leave a comment). To add one, set `old` to the document's first line and `new` to the block plus that line:

```yaml
---
numbering:
  headings:
    - {format: "1", after: "."}
    - {format: "1", parents: true}
  lists:
    - {format: "a", before: "(", after: ")"}
  series:
    schedule: {prefix: "Schedule"}
---
```

`headings` turns heading numbers on and gives each level's look (add `numbered: false` to give levels with numbers off). A level's `format` is `1`, `01`, `I`, `i`, `A`, `a` or `none`; `parents: true` shows the numbers above it, joined by `separator` (default `.`). `lists` applies to the whole document, and a single list can still override it; its formats add `bullet`. Levels past the last go on as the last.

With no `lists`, a new document's list under a numbered heading counts `(a)`, `(i)`, `(A)` by depth, and one under an unnumbered heading or none counts `1.`, `a.`, `i.`. An older document's block says what it has. Never rely on this when you cite a number: the numbers come from the comments.

A new block whose text starts with a typed number, such as `(a) `, `5.1 ` or `2. `, is refused with `typed_number`, and so is a marker other than `1.`. A plain number and a word is a sentence: `210 calls` is fine.

## `open_review` result

`review`, `version`, `format_version`, `role` (the user's: `owner`, `editor`, `suggester` or `viewer`), `owner` (whose document it is), `handoff?`, `mine: {open_threads, pending_suggestions, urls}`, `blocks: {ref: markdown}`, `working: [{by, since, lastSeen, stale}]`, `warnings`, and:

```
agenda.replies   [{thread, url, block, quote, new: [comment], needs_reply}]
agenda.threads   [{thread, url, block, quote, by, body, for_you}]
agenda.decided   [{suggestion, url, decision, by, was}]
agenda.edited    [{block, by, diff, protect: [string]}]
agenda.resolved  [{thread, url, by}]
```

The agenda covers what is new since your last clean submit; a partial failure brings the same agenda back. `mine` comes from current state.

## `submit_review` result

`version`, `results: [{key?, index, status, created?, urls?, reason?, code?, block_now?, details?, note?}]`, `changed: {ref: markdown}` (with `numbering` when the numbering block changed), `also_changed?: {ref: markdown}`, `also_changed_since?`, `warnings`, `notes?`.

Use `changed` and `also_changed` as your current text. `warnings` lists problems your change introduced, such as a dangling reference (`dangling-ref`) or a diagram that doesn't parse (`invalid-diagram`); the change still applied, so fix them yourself, in this round or the next. The `warnings` of `open_review` and `read_document` are every current problem, `{block, kind, target?, line?, error?, message}`: tell the user about those you didn't cause, and leave them unless asked. Common failure codes:

| code | do |
|---|---|
| `old_not_found` | Read `block_now`. If someone changed the text, redo your edit on it and keep their words. |
| `old_ambiguous` | Add surrounding words. |
| `markup_altered` | You changed a suggestion span or a comment marker. Copy them as they are, or choose an `old` that avoids them. |
| `invalid_link` | Use an `http`, `https` or `mailto` address, `codoc:{docId}` / `codoc:{docId}#label` for a Codoc document, or `#label` here. For a block here, `[](#label){.live}` shows its number and `[words](#label)` is a link: a plain `[](#label)` is neither. |
| `invalid_diagram` | A Mermaid diagram you wrote doesn't parse: fix the line in `details` (often a `;` in message text) and resend. See *Mermaid diagrams*. |
| `typed_number` | Write `1. ` for a list item, or an untitled heading line. |
| `outdated_plugin` | The server expects a newer plugin than this skill. Tell the user to update the plugin (`/plugin` in Claude Code) and stop; don't work around it. |
| `misplaced_id` | A `{#…}` is where no id can go, and would show as text. Put it alone on the line directly above the block, on a list item's first line, or at the end of a heading. If your `old` stopped just before a heading's id, include the id in `old` and `new`. To show it as words, write `\{#`. |
| `structural_not_suggestible` | Comment instead, or ask the user for direct edits. |
| `unread_comments` | Read the new comments before resolving. |
| `review_closed` | `open_review`, then resubmit. |

## A new document's Markdown

`create_document`'s `markdown` is read the same way, with every block new, so it holds only plain content:

- headings, paragraphs, lists (`1.` and `-`), tables, fenced code (including ```` ```mermaid ````), inline code, links, and images already uploaded;
- headings from `#`: the title is not in the text, so the top-level sections are heading 1 and their subsections heading 2, 3 and so on;
- `{#label}` on any block you will refer to, each label once; headings get one from their words if they have none (`cidr-ledger`, then `-2`, `-3`); `[](#label){.live}` and `[words](#label)` only to a block in the same text;
- the numbering block of a `md-source` export, if you start from one, is kept.

Markdown written elsewhere is turned into this first, and every change is listed in the result's `report`:

| written | becomes | `change` |
|---|---|---|
| `# Same as the title` at the very start | dropped | `title_dropped` |
| sections all `##` or deeper | moved up so the top level is `#` | `headings_promoted` |
| a heading without `{#…}` | labelled from its words | `label_assigned` |
| `## 2. Scope`, `### 4.1 Fees` | `Scope`, `Fees`: numbers are computed | `heading_number_removed` |
| `[w](../a.md)` with `base_url` | `[w](https://…/a.md)` | `link_resolved` |
| `[w](../a.md)` without `base_url`, or a scheme other than http, https, mailto | `w` | `link_kept_as_text` |
| `[w](#a-heading)` | a link in those words to that heading | `anchor_to_link` |
| `[w](#nothing-here)` | `w` | `anchor_kept_as_text` |
| `![alt](picture.png)` | a link to it (upload it to show it) | `image_kept_as_link` |
| two spaces, `\` or `<br>` at a line's end | a space: a paragraph is one run of text | `line_break_joined` |
| block quotes, footnotes, task boxes, raw HTML | kept as text | `kept_as_text` |
| `{#…}` where no id can go | kept as text. In a draft of your own this is a mistake: fix it with an edit | `kept_as_text` |
| `[]{#x}` at the start of a paragraph | the paragraph named `x` | none |
| HTML comments, `---` rules, YAML front matter | dropped | `dropped` |

Also, without a report entry because nothing a reader sees changes: `1. 2. 3.` lists are read as numbered lists, `_x_` and `__x__` as `*x*` and `**x**`, `~~~` fences as ```` ``` ````, setext headings as `#`. Code is never touched.

Refused, with nothing created: suggestion spans, comment markers and block comments (`markup_altered`), a typed number at the start of a paragraph (`typed_number`), a reference to a block that isn't there (`unknown_ref`, with a did-you-mean), an image `src` never uploaded (`invalid_image`), a Mermaid diagram that doesn't parse (`invalid_diagram`, naming which diagram, the heading it is under and the line), `numbering="…"` naming no series (`invalid_attribute`), a `base_url` or `source.url` that isn't https (`invalid`), and more than 128 KiB of Markdown (`invalid`).

## `create_document` result

`doc`, `url`, `version` (0), `title`, `labels: [{id, name}]`, `report: [{change, from, to?, count?, note?}]` (one entry per kind of change and text, with `count` when it happened more than once; empty for Codoc's own Markdown). The `url` opens it in Codoc for anyone who may read it; showing it is safe. The same `request_key` gives back the same `doc`, at its current `version`.

## Label results

`list_labels`: `labels: [{id, name, description, colour?, documents}]`. `update_label`: the label as it now is. `delete_label`: `{label, removedFrom}`.

| code | do |
|---|---|
| `label_duplicate` | A label with that name, or nearly, exists: `details.existing` is its id. Use it. |
| `unknown_label` | The message lists the labels. A label you deleted is gone. |
| `label_in_use` | `delete_label`'s `documents` was not the real number, which is in `details.documents`. Tell the user before sending it. |
| `forbidden` | A built-in label, which can't be deleted. |

## `read_document` result

`markdown`, `version`, `format_version`, `role`, `owner`, `source` (`{url, commit?}`, if it was imported with one), `working`, `warnings`, and `brief: {threads, suggestions, changes: {blocks, decided, resolved}, numbers, warnings}`. It never moves your cursor.
