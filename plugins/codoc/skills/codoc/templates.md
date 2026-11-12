# Templates

A document with variables is a template. Someone fills it in to make a document or a file, and the template itself never changes.

## Reading one

`read_document` shows the variables in a `variables:` block at the top, beside `numbering:`, each by its key:

```yaml
variables:
  client-name: {name: Client name, kind: text, default: Acme Ltd}
  company-logo: {name: Company logo, kind: image, ratio: "3:1", min: 600x200}
  plan: {name: Plan, kind: choice, options: [Starter, Pro, Enterprise], default: Starter, optional: true}
```

In the text, `{{client-name}}` is where a text variable goes, `![alt](codoc-var:company-logo)` an image, and `{when="plan: Pro"}` on a block or heading is a part shown only for those options.

The block is read-only in an edit: change variables with the tools below. Placeholders and conditions in the text are ordinary edits, and one naming a variable the document doesn't have is refused `unknown_variable`.

## Changing its variables

- **Add or change one** with `set_variable(doc, key?, name?, kind?, default?, optional?, options?, ratio?, min?)`. Leave out `key` to add one: its key is made from `name`, and `name` and `kind` are then needed. A variable's kind never changes.
- **A Choice's options** go in order, by name. `{name, was}` renames one, and its conditions follow. Leaving an option out removes it in the same change: it goes from every condition naming it, a condition left with none is removed so its part always shows, and a default that was that option goes too.
- **Optional** means it may be left empty and takes its default, so it needs one.
- **An image** has a `ratio` such as `3:1` (or `any`) and a `min` size such as `600x200`. Its default is an image's `src` from `upload_image`.
- **Delete one** with `delete_variable(doc, key)`. Its placeholders become its default, or go when it has none, and its conditions go, so their parts always show. The answer lists both. History can undo it.

## Making a document or a file from it

`create_from_template(doc, values, format, adjust?, title?, orgAccess?, request_key)`:

1. **Gather a value for each variable, by key:** a line of text, an option's name, or an image's `src`. An optional variable you leave out takes its default.
2. **Send each image first** with `upload_image` and target `{for: "value", doc}`, the template's id. It is placed nowhere; its answer's `src` is the value.
3. **An image of the wrong shape:** `adjust_image(doc, key, src, ...)` fits it to the variable's ratio and shows you the result. Look at it before you use it. Or pass `adjust` by key in the create call: `crop` (centred, the default), `fit` (transparent bars) or `stretch`.
4. **Choose `format`:** `codoc` for a new Codoc document, an export format for a file, or `gdoc` for a Google Doc in the user's Google Drive. A file comes back as `export` answers it: read `exports.md`.
5. **Send a `request_key`** you make up for this one creation, so a retry returns the same document. Pass `orgAccess` only when the user asked for it to be shared.
6. **A refusal for its values** lists what is `missing` and `invalid` in `details`, each by key and name. Fill those in and send it again.

To let the person fill it in themselves in one of their windows, use `show` with `fill` (`windows.md`). They press Create there.
