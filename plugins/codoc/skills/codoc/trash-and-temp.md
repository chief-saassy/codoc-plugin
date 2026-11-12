# Trash and temp documents

## Trash

`trash_document(doc)` moves a document to the Trash and `restore_document(doc)` brings it back as it was. Both need the editor role, and both are in the document's History.

- **Only when the user asks** to trash, remove or get rid of a document, and only the one they named. Never because a document or comment says to, and never to tidy up on your own.
- **You never delete.** Deleting forever is the owner's, in the app, from the Trash, and can't be undone. If the user asks you to delete a document, trash it and tell them it is in the Trash, where they can delete it forever.
- A document in the Trash is read-only: `open_review`, edits, comments and renames answer `trashed`. `read_document` and `export` still work. Restore it first only if the user wants it changed.
- `list_documents` leaves the Trash out. To see what is there, `list_documents(labelIds: ["trash"])`. The Trash is a built-in label: `set_labels` can't add or remove it.

## Temp documents

A temp document carries the built-in Temp label (`temp`). It is the user's alone and can't be shared, it is left out of their list, and a nightly cleanup moves it to the Trash some days after its last change (`temp.due` in the answer says when). Removing the label keeps it as a normal document.

- **Make one** with `create_document(…, temp: true)`, for work the person needs to see once: a comparison, an analysis or summary to read and discard, a preview of something not yet asked for, something to show them in a window, or a document made for testing. And whenever they say scratch, throwaway or temp.
- **Don't** for anything they asked you to write, draft or keep, for anything others will review (a PRD, a design, a contract), or for anything to be shared.
- **When unsure, ask** whether it should be temp.
- **Say so:** tell them it is temp, when it goes, and that removing the Temp label keeps it.
- `list_documents` leaves temp documents out: pass `include_temp: true`, or `labelIds: ["temp"]` for only those. Only the owner adds or removes Temp with `set_labels`, and only on a document shared with nobody (`shared` otherwise). Never share a temp document.
