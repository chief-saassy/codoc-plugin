---
name: prd
description: Use when writing, drafting or revising a PRD (product requirements document) in Codoc, and before a design document for important new or reshaped functionality people will see, such as a new screen, menu, flow or way of working. A PRD says what people get and why, with mocks; the design document that follows says how it is built.
---

# PRDs

A PRD says what a feature does for the people who use it and why, in enough detail that the owner can approve it and a builder can design from it. It says nothing about how it is built: no components, data model, code or races. Those belong in the design document that follows it (`design-doc` skill), which links back here.

## The project's own guidance comes first

This skill is the general base. Most projects have their own rules for PRDs, product design or UI, and those win where they differ.

- **Find them before writing.** Look through the skills available in this session for any on PRDs, specs, product or UI design, and read those that apply. Look in the repository too, if there is one: skill folders (`.claude/skills/`, and the like for other agents), `CLAUDE.md`, `AGENTS.md`, contributing guides, and design, UX or product docs. If you find none, say so and use this skill alone.
- **Follow what they say about content:** sections, wording, design principles, the UI kit, how decisions are recorded.
- **Bring them into Codoc.** Where a project skill writes the document to a file, write it in Codoc instead, unless the user or the project needs the file; then also export it there (`md-portable`, which carries the document's stamp) and keep the Codoc document as the one people review. Its template becomes the document's headings. Its review or approval steps happen as Codoc comments and replies. Its checklist goes into the reader's brief (*Reader check*).

## When one is needed

- **Needed** for important functionality people will see: a new screen, menu, panel or flow, a new kind of thing people create or manage, or a change to how people already work.
- **Not needed** for a fix, a small change to an existing control, or work people don't see. Those go straight to a design document, or to no document at all.
- When unsure, ask the user.

Write the PRD first, and get it approved before the design document. If a design document is asked for and the change needs a PRD that doesn't exist, say so and write the PRD first.

## Before writing

Find out what the user wants and why: the moment they wish it existed, who else is involved, and what would make it a failure even if it worked. Look at how the product does the nearest thing today, in the code and in the running product, so the feature fits what is there: the same controls in the same places, the same words for the same things.

## What it holds

In this order. Name each section for what it says.

1. **The summary, on the first screen,** under a heading of its own, such as `# Summary`: a document never opens with a bare paragraph. What people get, in plain words, and why it matters to them. Then how many decisions wait for the reader, each a tick box in the section it is about (*Decisions are tick boxes*).
2. **Who and when:** one or two concrete scenarios in the user's own terms ("a contract goes out as V1, comes back with redlines, and goes out again as V2"). Name the people involved and what each may do.
3. **What it does, as people meet it:** each flow from where it starts to what they see after, as numbered steps (*A process is numbered steps*). Every rule they will run into (who may, when it is refused, limits) and what a refusal says.
4. **The states off the happy path:** what people see when there is nothing yet, when there are very many, when they are offline or it is slow, and when someone else changes or removes the thing while they look. Leave out those that can't happen.
5. **Mocks** for every new or changed screen, menu, dialog or state that looks different, each beside the flow it shows, with a one-line caption. Draw them on the product's real look: the running product with the change added, or its own UI kit and icons, never a generic wireframe kit. Show the phone too when the layout differs there.
6. **Goals and non-goals:** what this must do, and what it deliberately doesn't. Non-goals stop the design growing; name anything the user ruled out.
7. **How we'll know it works:** the scenarios as checks a person could run in the product.
8. **A last line naming what was left out,** so the reader can ask.

Mark who decided what: *(owner)* for the user's decisions, *(writer's call)* for yours, so the reader can check yours. Where a question is still unsettled, mark it *(open)* where it comes up, with its decision there.

## A process is numbered steps

Whenever the text says what happens in turn (a flow a person goes through, what the product does after they act, what happens when someone else joins), write it as a numbered list:

- One step per item, in the order it happens, each starting with who acts: the person, another person, the product.
- A step that branches says where each answer goes, in the step itself: "If the name is taken, the dialog says so and keeps what they typed."
- A line before the list says what the steps achieve; a line after it, only if needed, says where the person ends up.
- A mock goes beside the step it shows.

You know you are writing a process when the sentences only make sense in their order, or when "then", "next" or "once" link them. A rule that holds at every step (who may, a limit) stays a sentence or a bullet beside the list.

A paragraph that reads one step after another:

```markdown
The owner presses Share and picks a person, who gets an email; once they open
the link they see the document as a viewer, and the owner can then raise them
to editor from the same dialog, which tells the person by email too.
```

The same as steps:

```markdown
1. The owner presses **Share** and picks a person.
2. The person gets an email with a link.
3. They open it and see the document as a viewer.
4. The owner can raise them to editor from the same dialog; the person gets a second email.
```

## Decisions are tick boxes

Each question only the reader can answer, and each call of yours they should check, is a decision written as the `codoc` skill's *Choices are tick boxes* says: one answer you recommend with an `Approve` box, or a real pick as "Tick one:". In a PRD:

- **Each decision sits in the section it decides**, straight after the information it depends on, so the reader approves once they have read what is being done. Never gather them in the summary; the summary says how many wait.
- **Ticked boxes stay** as the record of what was approved, and a ticked question is settled: build on it.

## Revising

Once the user has approved the PRD, it can still change, most often when designing or building turns up something it didn't foresee. When it does, ask yourself whether the change needs the user's approval before the work goes on. A change to what people get does; wording that changes nothing they get doesn't. *(writer's call)* is for choices the approved PRD leaves open, never a way around that approval.

## Keep it plain

Say what isn't obvious and stop. A mock carries no more words than the real screen would: a dialog that only asks for a name shows only the field. Words on screen are written as they will read. Spell out what people could lose or be surprised by (what happens to their work, whether it can be undone), and leave out what anyone would assume.

## Length

Be concise, but never leave out an important part of what the feature does: completeness on what matters comes before length. Don't count words. As a guide, an ordinary feature fits in about three pages, mocks not counted. Past that, split it into PRDs a reader can approve one at a time, or ask the user which part matters first, rather than going deeper.

## Reader check

Before handing over, have a fresh reader review the PRD, and finish that review yourself, so the user gets a finished document and only the questions that are truly theirs.

1. **Start a fresh subagent** that knows nothing of the conversation. Give it the document's url and this brief: read it as the person who must approve it, and as the builder who must design from it; leave a comment, with the hat `reader`, on each place where you would have to ask or guess and a wrong guess would matter, and on each process told as a paragraph instead of numbered steps; nothing you could settle yourself, and no wording nitpicks. Each round is a full review: read the whole document again, and new comments are expected.
2. **Answer every comment yourself.** If the conversation or the code answers it, write the answer into the document. If it is minor, decide it and mark it *(writer's call)*. Then reply saying what you changed, with links to where (`codoc` skill, *In a comment or a reply*), and resolve the reader's thread: this skill asks you to. A thread a person started, such as the user's own comment, gets the same answer and a reply but stays open: they resolve it, and you resolve it only if the user asks you to. Only a question the user alone can answer stays open: reply that it waits for them, mark it *(open)* in the text, and give it a decision with an Approve box there.
3. **Send the reader back** (the same subagent, whose agenda shows your replies and changes) until it leaves no new comment, at most three rounds.
4. **Hand over** with the open questions, each linked to its comment. Never hand over with a reader's comment unanswered.

Skip the check only for a one-page change the user asked for in a hurry.

## In Codoc

Use the `codoc` skill for the tools. Create the document with `create_document`, with a `request_key` and labels. Call `list_labels` first; a PRD gets the PRD label (such as `prd`) and the project's label, where they exist. Add each mock to the document as an image (the `codoc` skill's `files-and-assets.md`), then put it in place with an edit.

**A PRD and its design documents link to each other.** When a design document is written, add a link to it on the line after the PRD's summary ("Design: [its title](its url)"), and the design document links back the same way. A PRD with several design documents links to each one. Keep the links when either is revised.
