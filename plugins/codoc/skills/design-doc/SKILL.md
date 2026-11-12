---
name: design-doc
description: Use when writing, drafting or revising a design document, design note or technical explanation in Codoc, including explaining how an existing system works for someone who must change it or decide about it. Important functionality people will see needs an approved PRD first (prd skill); the design document then says how it is built.
---

# Design documents

Read everything you need. Write only what changes the reader's understanding or decision. Thoroughness shows up as correct claims, not as length.

## The project's own guidance comes first

This skill is the general base. Most projects have their own rules for design documents and their own design principles, and those win where they differ.

- **Find them before writing.** Look through the skills available in this session for any on design documents, specs, architecture or planning, and read those that apply. Look in the repository too: skill folders (`.claude/skills/`, and the like for other agents), `CLAUDE.md`, `AGENTS.md`, contributing guides, architecture or design docs, and ADRs. If you find none, say so and use this skill alone.
- **Follow what they say about content:** sections, design principles, how decisions and risks are recorded, which diagrams.
- **Bring them into Codoc.** Where a project skill writes the document to a file, write it in Codoc instead, unless the user or the project needs the file; then also export it there (`md-portable`, which carries the document's stamp) and keep the Codoc document as the one people review. Its template becomes the document's headings. Its review or approval steps happen as Codoc comments and replies. Its checklist goes into the reader's brief (*Reader check*).

## The PRD comes first

Important functionality people will see (a new screen, menu or flow, a new kind of thing people create or manage, a change to how people work) needs a PRD before its design document: what people get, why, the flows and rules they meet, and mocks (`prd` skill). If there is none, say so and write the PRD first, and get it approved.

The design document then covers how it is built: components, where state lives, data and events, what passes between components, who may read and write each new piece of data and what enforces it, races, migration, risks and tests. It doesn't repeat the PRD's behaviour, mocks or reasons; it links to them. Where the build forces a change to what people see, raise it against the PRD and change the PRD, so the two never disagree.

**They link to each other.** The design document's first line under its Summary heading is "PRD: [its title](its url)", and the PRD gets "Design: [this title](this url)" after its summary. Keep both links when either is revised.

A fix, a small change, or work people don't see needs no PRD.

## Before writing

Read the code, including how it fails, and what it does rather than what its comments say. Then settle these and tell the user in your message; they stay out of the document:

- **Reader:** who, and what they must decide or understand.
- **Size**, as a guide to depth, not a limit to measure against. Be concise, but never leave out an important part of the design: completeness on what matters comes before length. Don't count words. The size tells you how deep into detail to go before you split out another design document or ask the reader what they need. As a rough guide, at 500 words a page, diagrams not counted:
  - A decision or a small change: about 1 page.
  - An ordinary design, or an existing system explained: about 3 pages.
  - Something large: say why it needs more. If it would be too difficult to review at once, split it (see "Splitting a large design").
- **Level:** the level the decision is made at. Usually that is components, what each is responsible for, and where state lives. Go lower only where the decision turns on the detail.

## Splitting a large design

A design stays in one document if a reviewer can get through it in one go, however many components it has. Split it when both of these hold after cutting:

- It is still well over 3 pages, too much to review at once.
- Its components can each be reviewed on their own, once the reader knows how they fit together.

If its parts can't be understood apart, keep one document and say why it needs the length. If you split, tell the user why, and name the documents, in your message.

The split has two levels:

- **The master document** shows how the components fit together: the diagram, what each component is responsible for, where state lives, what passes between components, and the decisions that cross them. It links to each sub design and leaves the inside of each component to it.
- **A sub design** holds the detail of one component, for a component the master can't cover in a few lines. It links to the master and states only what is particular to that component, linking to the master or a sibling for the rest.

Each of these is a design document in its own right. Every rule in this skill, and every rule of the system it is written in, applies to each one separately: its own reader, size from the list above, summary on the first screen, diagram, code links, PRD link and "Left out" line. The one difference: a reader of a sub design has the master too, so the two together must be enough to decide or start work.

## What the document holds

In this order. Name each section for what it says, so the headings differ from task to task.

1. **The summary, on the first screen:** about 150 words under a heading of its own, such as `# Summary`: a document never opens with a bare paragraph. The recommendation or main finding comes first, in plain words, including what is surprising and what looks wrong: "no single object holds this state". Then what needs the reader: how many decisions wait (each in its own section, below), risks, what can't be undone, what is unverified.
2. **A diagram, if the document describes or changes a system.** At least a dependency diagram of about ten boxes, readable at a glance on a tablet. Merge boxes until it is. Flowcharts run top to bottom (`flowchart TD`), not left to right, since a page is taller than it is wide. Add a sequence diagram only where *A process is steps or a sequence diagram* calls for one; three diagrams at most in all.
3. **The few things the reader needs.** Of many similar things, name those that matter and count the rest. For problems, the roots, with their symptoms as one line each. For a decision, the options with their real costs and your recommendation, written as open: the reader decides. For a design, name the one or two other ways a reviewer would expect it built, and why not, a line each. For new data, who may read and write it, and what enforces that (access rules, a server check), including any rule change and how it is deployed.
4. **A last line naming what was left out,** so they can ask: "Left out: the retry states, the layout code. Ask for any of these." Details stay in the code or come on request, so this line is the only appendix.

Stay long enough to be used: someone must be able to decide or start work from this alone.

## A process is steps or a sequence diagram

Whenever the text says what happens in turn (a request through its checks, a loop, a retry, what runs when something fails), it takes one of two shapes:

- **Numbered steps**, by default. One step per item, in the order it happens, each starting with what acts (the page, core, the workflow, the model). A step that branches says where each answer goes, in the step itself. A step the argument rests on carries its code link. A line before the list says what the steps achieve; a line after it, only if needed, what holds at the end.
- **A Mermaid `sequenceDiagram`**, when three or more parts pass messages back and forth and the order between them is the point: a loop between a workflow, a vendor and core, or a race between two writers. If its labels say everything, the diagram stands alone. If any step needs more (the checks, what fails and how, what is kept), the diagram starts with `autonumber` and numbered steps follow it: every step, one per arrow, with the same numbers, so the reader can go from either to the other. Keep the diagram to the process the design turns on, about a dozen messages; any other process in the document stays a list. A `;` in a message breaks the diagram; use a comma.

You know you are writing a process when the sentences only make sense in their order, or when "then", "next" or "once" link them. A rule that holds at every step (who may, a limit) stays a sentence or a bullet beside the steps.

A paragraph that reads one step after another:

```markdown
Core checks the runner's role, the agent's role and the model link. Then one
transaction reads `running/{agentId}` and creates it with the run record,
unless a lock is there whose `deadline` hasn't passed (`already_running`).
Then core starts the workflow with the run id and the agent as it stands.
```

The same as steps:

```markdown
Starting a run makes sure an agent runs once at a time per document:

1. Core checks, in order, stopping at the first that fails: the runner may suggest; the agent's
   role is at least `viewer`; the runner's model link offers the agent's model.
2. One transaction reads the lock `running/{agentId}`.
   - A lock whose `deadline` hasn't passed: the start is refused with `already_running`.
   - None, or an expired one: it writes the lock and the run record together.
3. Core starts the workflow with the run id and a copy of the agent, so a change mid-run doesn't reach it.
```

## Decisions are tick boxes

Each decision the reader must make is written as the `codoc` skill's *Choices are tick boxes* says: one answer you recommend with an `Approve` box, or a real pick as "Tick one:". In a design document:

- **Each decision sits in the section it decides**, straight after the information it depends on, so the reader approves once they have read what is being done. Never gather them in the summary; the summary says how many wait.
- **Ticked boxes stay** as the record of what was approved, and a ticked question is settled: build on it.

## Claims

The claims the argument rests on link to the code, about four links a page, with the code's name as the link text:

[`foldReview`](https://github.com/owner/repo/blob/<full commit sha>/src/review.ts#L10-L25)

Each is a permalink to one pushed commit, in whatever forge the project uses (GitHub, GitLab and so on). Read each target at that commit (`git show <sha>:<path>`) and check it says what you claim. Links are for checking, so each finding gets one, and the reader can follow the code from there.

What you read in the code is stated plainly. Mark the rest where it appears: "inferred", or "not verified" with what would verify it.

## Revising

After feedback, cut as well as add, so the document stays as easy to read as the first draft.

Once the user has approved the document, the design can still change, most often when building turns up something it didn't foresee. When it does, ask yourself whether the change needs the user's approval before you build on it. A change to what is built or how does; wording that changes nothing built doesn't. *(builder's call)* is for choices the approved design leaves open, never a way around that approval.

## Before handing over

For each section, ask whether the reader's decision would change without it, and cut it if not. Then read the "Left out" line: anything there that would change the decision, or that a builder would need, moves into the document. If that makes it too deep to review at once, split it or ask the reader, rather than leaving it out.

## Reader check

Then have a fresh reader review the document, and finish that review yourself, so the user gets a finished document and only the questions that are truly theirs.

1. **Start a fresh subagent** that knows nothing of the conversation. Give it the document's url (and the PRD's, if there is one) and this brief: read it as the builder who must plan the work from it, checking its claims in the code; leave a comment, with the hat `reader`, on each place where you would have to ask or guess and a wrong guess would matter, on each claim the code doesn't bear out, and on each process told as a paragraph instead of numbered steps or a sequence diagram; nothing you could settle yourself, and no wording nitpicks. Each round is a full review: read the whole document again, and new comments are expected.
2. **Answer every comment yourself.** If the conversation or the code answers it, write the answer into the document. If it is minor, decide it and mark it *(builder's call)*. Then reply saying what you changed, with links to where (`codoc` skill, *In a comment or a reply*), and resolve the reader's thread: this skill asks you to. A thread a person started, such as the user's own comment, gets the same answer and a reply but stays open: they resolve it, and you resolve it only if the user asks you to. Only a question the user alone can answer stays open: reply that it waits for them, mark it *(open)* in the text, and give it a decision with an Approve box there.
3. **Send the reader back** (the same subagent, whose agenda shows your replies and changes) until it leaves no new comment, at most three rounds.
4. **Hand over** with the open questions, each linked to its comment. Never hand over with a reader's comment unanswered.

Skip the check for a one-page note on a small change.

## In Codoc

Use the `codoc` skill for the tools. Create the document with `create_document`, with a `request_key` and labels. Call `list_labels` first; a design note gets the design-note label (such as `dd`) and the project's label, where they exist. Put the PRD link on the first line, and add the link back to the PRD with an edit. Diagrams are Mermaid code blocks with lines of 100 characters or fewer.

For a split design, create the master first, then each sub design with a link to the master's `url` and the same labels, then edit the master to link to each sub design's `url`.
