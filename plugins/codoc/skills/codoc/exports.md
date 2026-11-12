# Exports

`format` is one of:

| format | gives |
|---|---|
| `docx` | Word |
| `pdf` | PDF |
| `md-portable` | Markdown for anywhere else: a repository, another tool |
| `md-source` | Codoc's own Markdown, which comes back into Codoc as it was, template variables included |
| `gdoc` | a Google Doc in the user's Drive (*A Google Doc*, below) |

`export(doc, format)` gives the document as it stands: pending suggestions are never in the file (proposed text is left out, text proposed for deletion stays), and comments never are. If the result has `warnings`, tell the user what they say. To get a suggestion into the file, it has to be accepted first, and only the user can ask for that.

The result has `filename`, `mime_type`, `size` and `inline`, and the file comes with it as a resource block:

- `inline:"text"`: Markdown, in the block's `text`. Write it to the file you were asked to, under the path the user named, as it is.
- `inline:"blob"`: Word or PDF, as base64. Put the file where it's wanted the first way that works, and never paste base64 into a message or a document:
  1. If you can run commands, fetch it with `curl`, below.
  2. If your client saved it and gave you a path, copy that file.
  3. Otherwise decode the base64 to a file.
- `inline:"none"`: too big to come this way; `message` says so. Use `curl`, below. If you can't run commands, give the user `url`, which they open signed in.

Fetching: `curl -fsSL -o "<filename>" "<download_url>"`, then check the size against `size`. If the command is refused, ask the user to allow it rather than trying other ways.

To send it somewhere (a repository, a cloud folder, an email), save it first and attach that file, using `filename`. Only when the user asked for that.

**The stamp.** Every file carries `stamp`, `codoc:<doc>@<version>`: which document, at exactly which version. It is metadata only, never in the text a reader sees:

| Format | Where |
|---|---|
| Word | custom document property `codoc` (also `codoc.doc`, `codoc.version`), in `docProps/custom.xml` |
| PDF | Info dictionary: `Keywords`, and the custom key `Codoc` |
| Codoc Markdown (`md-source`) | front matter line `codoc: "codoc:<doc>@<version>"` |
| Markdown (`md-portable`) | first line `<!-- codoc:<doc>@<version> -->`, then a blank line |
| Google Doc | the Drive file's property `codoc`, set by the export and by a restamp |

To trace a file you are given, search its bytes for `codoc:` followed by `@` (e.g. `grep -ao 'codoc:[A-Za-z0-9_-]*@[0-9]*' file`); in Word and PDF it is stored uncompressed. Text extracted from Word or a PDF won't have it. If you make a file from a Codoc document yourself (your own formatting, a different converter), stamp it the same way, in the same place for its format: take `stamp` from `export`, or build it from `read_document`'s `doc` and `version`, the version your content was read at. Never put it in the visible text.

**A Google Doc.** `export(doc, "gdoc")` makes a Google Doc of the current version in the user's own Google Drive, converted from the Word file, and returns its `url` and `file_id`, not a file. `drive_not_connected` means they haven't connected Google Drive: tell them to connect it in Codoc's Settings (`/settings#google-drive`), then try again.

- **Check it, every time.** Codoc reads the Google Doc back and names what differs in `warnings`, each with the `file_id`, but its check is not yours. After every Google Doc export, warnings or not, open the Google Doc with Google's own tools (a Google Drive or Google Docs connector), compare it with the document (`read_document`), and fix by hand what the conversion got a little wrong: a number, a heading level, a table, a picture. Then `export(doc, "gdoc", restamp: <file_id>)`, so the Google Doc's stamp says which version it matches; that answer checks it again. If you have no Google tools, tell the user to look it over, and what `warnings` said.
- **One copy each.** If the user already has a Google Doc of this document, `export` makes nothing and answers `copy_exists`; `details.copies` names each with its version and `url`. Ask the user: bring one up to date, or make a new copy? A new copy is `new_copy: true`. To bring one up to date, change that Google Doc with Google's own tools to match the document as it stands, then `restamp: <file_id>`. Codoc restamps only a Google Doc it made for this user.

**`download_url` is a secret.** Anyone holding it can get that file for 15 minutes. Use it only to fetch; never put it in a document, comment, reply, handoff, commit or message to anyone, and don't show it unless the user asks. If it has expired, call `export` again. A `not_configured` answer on PDF means PDF isn't available here: say so, and offer Word.
