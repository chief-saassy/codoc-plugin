---
name: design-doc
description: Use when writing, drafting or revising a design document, design note or technical explanation in Codoc, including explaining how an existing system works for someone who must change it or decide about it.
---

# Design documents

Read everything you need. Write only what changes the reader's understanding or decision. Thoroughness shows up as correct claims, not as length.

## Before writing

Read the code, including how it fails, and what it does rather than what its comments say. Then settle these and tell the user in your message; they stay out of the document:

- **Reader:** who, and what they must decide or understand.
- **Budget**, at 500 words a page, diagrams not counted:
  - A decision or a small change: 1 page.
  - An ordinary design, or an existing system explained: 3 pages.
  - Something large: say why it needs more, and split it by decision into separate documents.
- **Level:** the level the decision is made at. Usually that is components, what each is responsible for, and where state lives. Go lower only where the decision turns on the detail.

## What the document holds

In this order. Name each section for what it says, so the headings differ from task to task.

1. **The conclusion, on the first screen:** about 150 words straight after the title, with no heading of its own. The recommendation or main finding comes first, in plain words, including what is surprising and what looks wrong: "no single object holds this state". Then what needs the reader: decisions, risks, what can't be undone, what is unverified.
2. **A diagram, if the document describes or changes a system.** At least a dependency diagram of about ten boxes, readable at a glance on a tablet. Merge boxes until it is. One or two diagrams in all.
3. **The few things the reader needs.** Of many similar things, name those that matter and count the rest. For problems, the roots, with their symptoms as one line each. For a decision, the options with their real costs and your recommendation, written as open: the reader decides.
4. **A last line naming what was left out,** so they can ask: "Left out: the retry states, the layout code. Ask for any of these." Details stay in the code or come on request, so this line is the only appendix.

Stay long enough to be used: someone must be able to decide or start work from this alone.

## Claims

The claims the argument rests on link to the code, about four links a page, with the code's name as the link text:

[`foldReview`](https://github.com/owner/repo/blob/<full commit sha>/src/review.ts#L10-L25)

Each is a GitHub permalink to one pushed commit. Read each target at that commit (`git show <sha>:<path>`) and check it says what you claim. Links are for checking, so each finding gets one, and the reader can follow the code from there.

What you read in the code is stated plainly. Mark the rest where it appears: "inferred", or "not verified" with what would verify it.

## Revising

After feedback, cut as well as add. The budget still holds unless the reader changes it.

## Before handing over

Count the words, leaving out diagrams. Over budget means cutting, not raising the budget. For each section, ask whether the reader's decision would change without it, and cut it if not. Then read the "Left out" line: anything there that would change the decision moves into the document, using what the budget has left. Tell the user the count against the budget.

## In Codoc

Use the `codoc` skill for the tools. Create the document with `create_document`, with a `request_key` and labels. Call `list_labels` first; a design note gets the design-note label (such as `dd`) and the project's label, where they exist. Diagrams are Mermaid code blocks with lines of 100 characters or fewer.
