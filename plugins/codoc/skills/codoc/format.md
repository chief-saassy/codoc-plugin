# Codoc format reference

Read this when `SKILL.md` doesn't settle a question about the text you were given.

## Agent Markdown

Clean Markdown, one sentence per line, plus:

| you see | it is | you may |
|---|---|---|
| `{#sec:fees}` after a heading | the block's handle | use it as `block` and in `@sec:fees` |
| `<!-- 1.1 -->`, `<!-- (b), cited as 1.1(b) -->` | a computed number, and for a list item how to cite it | quote it to people; it is ignored on write |
| `@sec:fees <!-- clause 1.1 -->` | a reference | write `@sec:fees`; the number follows |
| `{++new++}{>>#s13 @sam/claude<<}` | suggested insertion | copy as is |
| `{--old--}{>>#s14 @adrian<<}` | suggested deletion | copy as is |
| `{~~old~>new~~}{>>#s12 @adrian<<}` | suggested replacement | copy as is |
| `{--60--}{>>#s16 @sam/claude of #s12<<}` | a deletion of text `s12` proposed | copy as is |
| `{c7>}`, `{<c7}` | where thread `c7` is anchored | copy as is; can't be moved |
| `{c8}` on its own line | a thread on the whole block below | copy as is |
| `{>>#c7 @adrian 2026-10-24: …<<}` | one comment, read-only | use a `reply` item |
| `[words](https://…)`, `[words](codoc:abc#sec:fees)` | a link; `codoc:` is another Codoc document | keep it as it is; a bad address is refused with `invalid_link` |
| `\*`, `\{`, `\@sec:` | a backslash escape: plain text, never markup | copy it in `old`; don't add one in your own `new` |

An author reads `@user`, then `/client` for anything but the web editor, then `/hat`: `@sam/claude-code/security`. Decided suggestions and resolved threads aren't shown; `thread(doc, thread)` has their history.

## Changing text that already carries suggestions

Change someone else's proposed text and you make your own suggestion on top of theirs. Say `{~~30~>60~~}{>>#s12 @adrian<<}` should be 45:

```json
{"edit":{"block":"sec:fees-due","old":"{~~30~>60~~}","new":"{~~30~>45~~}"}}
```

The tag stays out of `old`, and the result reads `{--30--}{>>#s12 @adrian<<}{--60--}{>>#s16 @sam/claude of #s12<<}{++45++}{>>#s16 @sam/claude<<}`. Changing your own proposed text amends it. Deleting your own proposed text removes it.

One edit that changes someone's proposed text and plain text as well is still one suggestion: it depends on theirs as a whole, so rejecting theirs rejects all of it. To keep the plain change independent, send it as its own edit. One suggestion can show several marked spans, each followed by the same tag.

## Whole-block replacement

`{"edit":{"block":"sec:fees","text":"…"}}` replaces the block. Send it as you read it, with every tag, marker and thread line intact, and with your changes as plain text. It is refused if anyone else changed the block since you read it. Deleting a paragraph may take its thread lines with it; those threads become orphaned.

## Images

An image is a line `![alt text](codoc-image:sha256-…)` with its caption on the lines after it, up to a blank line. To add one, call `upload_image(doc, data)` first (base64 of the file, or the text of an SVG with `encoding: "utf8"`), then put the `markdown` line it returns, followed by a caption, in an edit, with a blank line before and after. Uploading the same bytes again gives the same `src`. The caption is text like any other; changing the alt text or the picture is a direct edit, refused in suggest mode. A `src` that was never uploaded is refused.

## Structure attributes

Braces go at the end of a heading line, and straight after an item's marker. A new block gets a `#sec:` label you choose: lowercase, unique, descriptive.

| attribute | on | meaning |
|---|---|---|
| `#sec:label` | both | the handle |
| `numbered`, `-` | heading | number this heading or not, whatever the document's default |
| `numbering="schedule"` | heading | starts a series: "Schedule 1 — Services" |
| `empty` | untitled heading | the number stands alone; what follows starts below it |
| `start=3` | numbered heading | this heading counts from 3: it reads 3, the next at its level 4, and the levels inside it count as usual under it. `start=1` restarts the numbering here |
| `format="(a)"` | item | this list's look from here. `(a)`, `a.`, `A`, `(i)`, `i`, `I`, `1.`, `(1)`, `01`, `bullet` |
| `start=3` | item | the list starts at 3 |
| `continue` | item | carry on from the previous list, instead of restarting |

An empty normal paragraph reads `{#sec:x empty}`, which is the only way to say which block it is.

In this example, "This includes…" continues (a), and "pay the Fees" starts a new list:

```markdown
# Obligations {#sec:obligations} <!-- 3 -->

## {#sec:duty} <!-- 3.1 -->
The Supplier owes these duties.

The Supplier shall:

1. {#sec:skill-care format="(a)"} provide the Services with reasonable skill and care; <!-- (a), cited as 3.1(a) -->

   This includes keeping its staff trained.

   1. {#sec:staff format="(i)"} by annual courses; <!-- (i), cited as 3.1(a)(i) -->

1. {#sec:policies} comply with the Customer's policies. <!-- (b), cited as 3.1(b) -->

The Customer shall:

1. {#sec:pay format="(a)"} pay the Fees; <!-- (a), cited as 3.1(a) -->
```

Writing `{#sec:pay format="(a)" continue}` would make that (c).

Comments carry no meaning. `## {#sec:x} <!-- empty -->` is not how to say a number stands alone, and `<!-- 7 -->` doesn't set a number.

A new document's lists count `(a)`, `(i)`, `(A)` by depth unless the document or the list says otherwise. Never rely on this in a clause reference: the numbers come from the comments.

A new block whose text starts with a typed number, such as `(a) ` or `5.1 `, is refused with `typed_number`, and so is a marker other than `1.`.

## `open_review` result

`review`, `version`, `format_version`, `handoff?`, `mine: {open_threads, pending_suggestions}`, `blocks: {ref: markdown}`, `working: [{by, since, lastSeen, stale}]`, `warnings`, and:

```
agenda.replies   [{thread, block, quote, new: [comment], needs_reply}]
agenda.threads   [{thread, block, quote, by, body, for_you}]
agenda.decided   [{suggestion, decision, by, was}]
agenda.edited    [{block, by, diff, protect: [string]}]
agenda.resolved  [{thread, by}]
```

The agenda covers what is new since your last clean submit; a partial failure brings the same agenda back. `mine` comes from current state.

## `submit_review` result

`version`, `results: [{key?, index, status, created?, reason?, code?, block_now?, note?}]`, `changed: {ref: markdown}`, `also_changed?: {ref: markdown}`, `also_changed_since?`, `warnings`, `notes?`.

Use `changed` and `also_changed` as your current text. `warnings` lists problems your change introduced, such as a dangling reference; the change still applied. Common failure codes:

| code | do |
|---|---|
| `old_not_found` | Read `block_now`. If someone changed the text, redo your edit on it and keep their words. |
| `old_ambiguous` | Add surrounding words. |
| `markup_altered` | You changed a tag, marker or thread line. Copy them as they are, or choose an `old` that avoids them. |
| `invalid_link` | Use an `http`, `https` or `mailto` address. |
| `typed_number` | Write `1. ` for a list item, or an untitled heading line. |
| `structural_not_suggestible` | Comment instead, or ask the user for direct edits. |
| `unread_comments` | Read the new comments before resolving. |
| `review_closed` | `open_review`, then resubmit. |

## A new document's Markdown

`create_document`'s `markdown` is read the same way, with every block new, so it holds only plain content:

- headings, paragraphs, lists (`1.` and `-`), tables, fenced code (including ```` ```mermaid ````), links, and images already uploaded;
- `{#sec:label}` on what you will refer to, each label once; `@sec:label` only to a block in the same text;
- the numbering front matter of a `md-source` export, if you start from one, is kept.

Refused, with nothing created: CriticMarkup, suggestion tags, comment markers, block comments and thread lines (`markup_altered`), typed numbers (`typed_number`), a reference to a block that isn't there (`unknown_ref`, with a did-you-mean), an image `src` never uploaded (`invalid_image`), `numbering="…"` naming no series (`invalid_attribute`), and more than 128 KiB of Markdown (`invalid`).

## `create_document` result

`doc`, `url`, `version` (0), `title`, `labels: [{id, name}]`. The `url` opens it in Codoc for anyone who may read it; showing it is safe. The same `request_key` gives back the same `doc`, at its current `version`.

## Label results

`list_labels`: `labels: [{id, name, description, colour?, documents}]`. `update_label`: the label as it now is. `delete_label`: `{label, removedFrom}`.

| code | do |
|---|---|
| `label_duplicate` | A label with that name, or nearly, exists: `details.existing` is its id. Use it. |
| `unknown_label` | The message lists the labels. A label you deleted is gone. |
| `label_in_use` | `delete_label`'s `documents` was not the real number, which is in `details.documents`. Tell the user before sending it. |
| `forbidden` | A built-in label, which can't be deleted. |

## `read_document` result

`markdown`, `version`, `format_version`, `working`, `warnings`, and `brief: {threads, suggestions, changes: {blocks, decided, resolved}, numbers, warnings}`. It never moves your cursor.
