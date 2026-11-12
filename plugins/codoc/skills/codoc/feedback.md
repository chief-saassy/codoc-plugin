# Feedback to the Codoc team

`send_feedback` tells the Codoc team about a problem with Codoc or an improvement. Send one:

- **when the person asks you to** (`initiative: "asked"`);
- **on your own initiative** (`initiative: "own"`), only about your own experience of Codoc's tools: a refusal you can't make sense of, an answer that contradicts this skill or the tool descriptions, something that should work and doesn't, an error you had to work around, or an improvement that would have made the work easier. Not about the person's documents, and not their opinions of Codoc unless they ask you to pass them on.

On your own initiative, call `feedback_setting()` first, and draft nothing when it says `off`. Change the setting only when the person asks (`feedback_setting(mode)`).

- **`off`**: nothing is recorded, even when the person asked. On `feedback_off`, tell them they can turn it on in Settings, under Agents, or use Send feedback in the app.
- **`ask`** (Ask me first, the default): show the person the report word for word and ask whether to send it. Only after they say yes to that exact text, send exactly that text with `approved: true`. If they change it, show it again. Without `approved`, the answer is `needs_approval`.
- **`send`** (Send without asking): send it, then tell the person in one line that you did.

Write the report so the team can reproduce the problem: the tool, what you sent, what came back (the code), and what you expected. Use **examples you make up** ("an edit whose `old` is `Alpha beta`"), never the person's own words, names, figures, document titles or ids, unless they asked you to include them. Pass `doc` and `hat` only when the person asked you to send feedback about that document.

**At most one report on your own initiative per problem per conversation.** Codoc can't count for you: it keeps no conversation. Several refusals of one kind are one problem.

Always send `plugin_version` (this skill's version, in its description) and a `request_key` you choose for this report. If the answer is lost, send the same call with the same key: it writes one report.
