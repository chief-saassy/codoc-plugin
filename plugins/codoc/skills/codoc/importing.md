# Importing into Codoc

## Importing existing Markdown

When the user asks to bring in a Markdown file (a README, a design doc from GitHub or GitLab), send it **unmodified**: don't rewrite links, renumber or reflow it first. Codoc does the conversion and tells you what it did.

- `title`: the document's name, usually the file's `# heading` (or the user's choice). A leading `# heading` that says the same as the title is dropped, so it isn't there twice; if the sections were then `##`, every heading moves up one so top-level sections are heading 1.
- `base_url`: the https address of the file as it is viewed (`https://gitlab.com/group/repo/-/blob/main/docs/design.md`), so relative links like `../terraform/AGENTS.md` resolve to working links. Without it a relative link is kept as its words and reported, never refused.
- `source`: `{url, commit?}`, where it came from (the same address, and the commit if you know it). It is kept with the document and `read_document` returns it.
- Links to `#a-heading` stay links to that heading; one matching nothing stays as its words. Hard-wrapped lines join into paragraphs, typed heading numbers (`## 2. Scope`) come off (Codoc numbers headings), a numbered list stays numbered, a line holding only `\newpage` or `\pagebreak` is kept as a page break, and what Codoc has no form for (block quotes, footnotes, raw HTML, horizontal rules) is kept as text or dropped, each listed.
- Then check `report` and tell the user, in a line or two, what changed that they would notice: links kept as text (and why), the title heading dropped, anything kept as text or dropped. Give them the `url`.

## Importing a Google Doc

When the user gives you a Google Docs link and asks to bring it into Codoc, call `create_document(title?, google_doc, labels, request_key)` with the link as they gave it (or its file id), not `markdown`. Leave out `title` unless they named it: it defaults to the Google Doc's name. Any Google Doc they can open in the Google account they connected in Codoc's Settings can be imported; comments and suggestions are not. The tab the link names comes in (or the first): the report gives each other tab's own link, to import as a document of its own. A Google Doc written as a template comes in as one: each `{{Name}}` in its text becomes a text variable with no default, the same name in any case being one variable (`placeholder_to_variable` in the report); braces in a link or in code stay words. Choices, conditions and image placeholders aren't read from a Google Doc: if the user wants them, add them in Codoc afterwards.

- Headings, lists in their own numbering ((a), (i), 1.…), bold, italic, tables, links and pictures come in; typed heading numbers ("1. Scope") come off, as Codoc numbers headings itself, and every number reads as it did in the Google Doc.
- Page breaks come in: a page break, a section break that starts a new page, and a paragraph set to *Add page break before* each become one. A page break inside a table cell is dropped and listed. Page ends where the text simply filled the page are not page breaks, so they don't.
- Then tell the user, in a line or two, what `report` says didn't come through as it was: other tabs, footnotes (kept as [1] with their words at the end), pictures left as `[picture: …]`, a list shown in another form, headers and footers. Give them the `url`.
- `drive_not_connected`: tell them to connect Google Drive in Settings (`/settings#google-drive`), then try again. `not_found` or `forbidden`: that Google account can't open the Google Doc; ask them to check the link or share it with that account.
