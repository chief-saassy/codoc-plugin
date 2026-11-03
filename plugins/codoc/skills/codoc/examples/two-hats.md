# Two hats on a design document

The user asks for a security review and an alternatives review of a token cache design, as one piece of work. Run one review per hat, typically one subagent each, and pass the same `group`.

## security

`open_review({doc:"d2", hat:"security"})`, then:

```json
{
  "review": "r1",
  "group": "design review",
  "done": true,
  "handoff": "Suggested in-memory-only caching; asked about key rotation.",
  "items": [
    {"edit":{"block":"sec:cache","old":"until it expires","new":"in memory, never in KV or logs, until it expires"}},
    {"comment":{"block":"sec:keys","quote":"service account key","body":"How is this key rotated, and who can read the secret?"}}
  ]
}
```

## alternatives

`open_review({doc:"d2", hat:"alternatives"})`. The agenda doesn't list security's threads, because they aren't addressed to this hat; the text does show them:

```markdown
The Worker caches the ID token per user {++in memory, never in KV or logs, ++}{>>#s1 @sam/claude-code/security<<}until it expires.
```

Its changes to security's proposed text become its own suggestion on top. Its comment may overlap security's anchor:

```json
{
  "review": "r2",
  "group": "design review",
  "done": true,
  "handoff": "Asked for a paragraph on the service-account option; widened s1 to cover D1.",
  "items": [
    {"edit":{"block":"sec:cache","old":"never in KV or logs","new":"never in KV, D1 or logs"}},
    {"comment":{"block":"sec:keys","quote":"service account","body":"This option deserves its own paragraph: what would keyless signing cost us?"}}
  ]
}
```

It does not decide, resolve or reply to anything of security's.

## Afterwards

The owner filters the review margin by hat and decides each review in turn. Only if asked, an `editor` hat merges the two: it reads both reviews and may decide suggestions, because the user asked it to.
