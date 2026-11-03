# A worked round: open, submit, partial failure, resubmit, done

A second round on a webhook design. Earlier, the reviewer suggested a staged rollout and commented on the logs. Sam rejected the rollout change and replied in both threads.

## Open

```json
{"doc":"d1","hat":"reviewer"}
```

Trimmed result:

```json
{
  "review": "r3",
  "version": 14,
  "role": "suggester",
  "owner": "sam",
  "handoff": "First pass done: suggestions on retries, signing and rollout, threads open.",
  "agenda": {
    "replies": [
      {"thread":"c1","block":"sec:storage","quote":"the customer API key","needs_reply":true,
       "new":[{"body":"Agreed. Please propose wording that stops logging the key."}]},
      {"thread":"c4","block":"sec:rollout","quote":"all customers","needs_reply":true,
       "new":[{"body":"Friday evening is the only window we have, so this stays as written."}]}
    ],
    "decided": [{"suggestion":"s3","decision":"reject","by":"sam","was":"replace 'Friday evening' with 'a Monday morning'"}],
    "edited": [{"block":"sec:monitoring","by":"sam","diff":"…","protect":["Paging goes to the on-call rota, as agreed with SRE on 14 October."]}],
    "threads": [], "resolved": []
  },
  "blocks": {
    "sec:storage": "# Delivery logs {#sec:storage -}\n\nDelivery logs keep the full payload and {c1>}the customer API key{<c1} for 90 days.\n{>>#c1 @alex/claude-code/reviewer 2026-10-20: API keys should not be in the logs at all.<<}\n{>>#c1 @sam 2026-10-20: Agreed. Please propose wording that stops logging the key.<<}"
  }
}
```

So: word c1 and reply, reply on c4 with no new suggestion, and leave `sec:monitoring` alone. The document is Sam's, not the user's, so the wording goes in as a suggestion: `mode:"suggest"`.

## Submit

The edit comes first, `old` avoids the comment markers, and the reply cites the edit by key.

```json
{
  "review": "r3",
  "mode": "suggest",
  "group": "webhook review",
  "done": true,
  "handoff": "Wording for c1 suggested; c4 acknowledged; rollout left as written.",
  "items": [
    {"key":"e1","edit":{"block":"sec:storage","old":"keep the full payload and","new":"keep the full payload, without","re":"c1"}},
    {"reply":{"thread":"c1","body":"Proposed in $e1."}},
    {"reply":{"thread":"c4","body":"Understood. It stays as written."}}
  ]
}
```

Sam changed that sentence as this arrived:

```json
{
  "version": 16,
  "results": [
    {"key":"e1","index":0,"status":"failed","code":"old_not_found",
     "reason":"'keep the full payload and' was changed since version 14 by sam; see how the block now reads",
     "block_now":"# Delivery logs {#sec:storage -}\n\nDelivery logs keep (per the data policy) the full payload and {c1>}the customer API key{<c1} for 90 days.\n…"},
    {"index":1,"status":"skipped","reason":"depends on item 'e1', which failed"},
    {"index":2,"status":"applied"}
  ]
}
```

The c4 reply is written, so it isn't sent again. Sam's words are new, so they stay. `done:true` closed the review, so a new one is needed.

## Resubmit

`open_review` with the same arguments returns `r4`. The agenda is the same, because the cursor moves only on a clean submit, but c4 now shows `needs_reply: false`. Send only the two items left:

```json
{
  "review": "r4",
  "mode": "suggest",
  "group": "webhook review",
  "done": true,
  "handoff": "Wording for c1 suggested; c4 acknowledged; rollout left as written.",
  "items": [
    {"key":"e1","edit":{"block":"sec:storage","old":"the full payload and","new":"the full payload, without","re":"c1"}},
    {"reply":{"thread":"c1","body":"Proposed in $e1. I kept your words on the data policy."}}
  ]
}
```

All items report `applied`, so the cursor moves. Tell the user what was suggested and answered, and that Sam's edit landed mid-round.
