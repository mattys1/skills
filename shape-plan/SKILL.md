---
name: shape-plan
description: >
  Turn a resolved spec into an implementation plan, discussing the shape with
  the user.
---

# Shape-plan

You take a spec whose decisions are already locked and produce an implementation plan
for it. Before writing the plan, you and the user agree on the **shape** of the code:
its types, signatures and seams. The shape is negotiated as illustrative code written
straight into the files where it will live, so the user can edit it and you can read
their edits back with `git diff`.

The flow has four phases, with an explicit handover between them:

0. **Gather**: load the spec and everything it rests on, then build the decision ledger.
1. **Propose the shape**: write it into the code and explain it in chat, with usage examples.
2. **Iterate**: read the user's edits, check them against the ledger, and revise until they agree.
3. **Plan**: write the implementation plan around the agreed shape, then stash the shapes.

Say out loud when you move from one phase to the next.

---

## Sources of truth

Things will disagree. Resolve them in this order, highest first:

1. **The spec's resolution.** The `## Resolution` of the issue (body or comments), plus
   any "Requirement from …" sections that other specs pushed into it.
2. **Changes agreed on the parent issue / map** (e.g. "Changes agreed while charting").
   They override older decision records they explicitly reverse.
3. **Linked spec issues** this one depends on, for whatever they decided that this spec
   builds on (e.g. the Shape store spec builds on the compression spec's `Imported` map).
4. **Domain docs:** `GLOSSARY.md` (or `GLOSSARY-MAP.md`) and `docs/adr/`. The
   glossary governs naming. Also any other decision record the spec or its parent
   explicitly links to (e.g. a design-notes file listing earlier decisions); don't go
   hunting for such files otherwise.
5. **The code as it is today.** It describes the before-state, not what should be built.
6. **Older material** (a thesis, old notes). Background only.

**The grilled decisions outrank the shape.** A shape exists to express the decisions,
not to revisit them. If any proposed shape, yours or the user's, contradicts a locked
decision, flag it and reject it (see Phase 2). The user can reopen a decision, but that
happens in a grilling session that updates the issue, not by editing a shape file.

### Docs are read-only here

Consume the domain docs as `docs/agents/domain.md` describes, if the repo has it: read
the glossary before you name anything, use its terms, read the ADRs that touch the area,
and flag ADR conflicts explicitly. But **do not edit** `GLOSSARY.md`, ADRs, linked
decision records or the issues. If shaping reveals a missing term or a decision the docs
don't cover, tell the user. Settling it is the job of a separate grilling session that
they run, not of this skill. Then carry on with what is settled.

---

## Phase 0: Gather

Read everything in the source list above, including the issue comments, the
code the spec touches, and the code rules in `CLAUDE.md` along with the docs in `GLOSSARY.MD`. The code rules constrain the
shape as much as the spec does. Three things to check while you read:

- **No resolution, or decisions still open:** stop. This skill needs locked decisions.
- **A prerequisite spec hasn't landed in the code yet:** say so. The shape then builds on
  that spec's agreed design, not on today's code.
- **A file you'll shape has uncommitted changes:** ask the user to commit or stash it
  first. Otherwise the final stash would take their changes with the shapes.

### The decision ledger

Distill the locked decisions that bear on the code into a numbered list, `L1`, `L2`, …
Each entry is one decision, quoted or closely paraphrased, with its source:

```
L1  EdgeData gains `shape: Nullable<ShapeID>`, mirroring `properties`. (#4 Reference from Edge)
L2  Only interior Shape points are stored; Node coordinates are not repeated. (#4 What is stored)
L3  Shape points are stored in from→to travel order; reversed for Reverse ways. (#4 Point order, #3 Direction)
L4  Graph doesn't know about the ShapeStore; Database owns it. (#4 Store)
...
```

Include behavioural decisions too (they become plan content later), but mark the ones
that pin down shape. Also list **gaps**: things the shape will need that the spec leaves
open. Gaps are fair game for shaping; contradictions are not.

Show the ledger and the gaps to the user in chat and keep the numbering stable for the
rest of the session. Every flag you raise later cites a ledger number.

---

## Phase 1: Propose the shape

### What goes into a shape

A shape is illustrative code. It does not have to compile. It should contain:

- data types: structs, enums, newtypes, with field types;
- public API: function and method signatures with `todo!()` (or the language's
  equivalent) bodies;
- trait impls, derives and visibility that matter to the design;
- the seams between components: who owns what, who calls whom, what crosses the boundary;
- private details only where the spec pins them down (a field layout, a const parameter);
- short doc comments that tie items to ledger entries (`// L3`) and explain intent;
- open points as `// Q:` comments.

Leave out algorithms and logic. That belongs in the plan.

Follow the codebase: its naming, module layout, error types, idioms. Use glossary terms
for every domain concept and never a term the glossary lists under _Avoid_.

### Where it goes

Write each piece where it would really live: edit the existing file in place, or create
the new file at its intended path. Mark every shaped region so it is easy to find and
remove:

```rust
// SHAPE(shape-store) ─────────────────────────────
pub(in crate::database) struct ShapeStore {
    pool: ClusterPool<ShapePoint, 4, ShapesFile>, // L5
}
// ───────────────────────────────── /SHAPE
```

Keep shapes minimal inside existing files. Show only what is new or changed, and leave
the rest of the file alone.

### What goes into chat

For each component:

1. **A short summary**: what it is, what it owns, which ledger items it realizes.
2. **The key code**, or a pointer to it (`src/database/shape.rs:12`) if it is long.
3. **Usage examples**: two to four short snippets showing how the shape is *used* from
   its real call sites, e.g. the importer filling it, the exporter reading it, a test
   asserting on it, `Database::save` persisting it. Usage is where awkward shapes show
   up, so make the examples realistic. Use the actual names of the surrounding code.

Then list any `// Q:` points and gaps you need the user to decide.

### Hand over the round

After writing, stage the shape files: `git add <files>` (`git add -N` is not enough
for new files; stage them fully). The index now holds your latest proposal, so the
user's edits show up as unstaged changes. Tell the user:

> Shape round 1 is in (staged). Edit the files directly or reply in chat. When you're
> done, tell me and I'll read your edits with `git diff`.

---

## Phase 2: Iterate

When the user says they have edited or replies with feedback:

1. **Read the edits.** `git diff` shows what the user changed since your last round.
   Also run `git status` for new untracked files. If the diff touches files you didn't
   shape, ask whether that is part of the shape.
2. **Check every change against the ledger.**
   - **Contradicts a locked decision → reject it.** Quote the ledger entry and explain
     the conflict in one or two sentences:
     > Rejecting the `shape: Vec<ShapePoint>` field in `EdgeData`: L1 says the Edge
     > holds `Nullable<ShapeID>` into a separate store. If you want to reopen that,
     > it's a grilling question for #4, not a shape change.

     Don't build on the rejected change. Leave the user's text in place, but don't let
     it shape later rounds or the plan. If the user wants to override, tell them the
     decision has to be reopened at its source: they can update the issue, or run a
     grilling session. You don't change the issue yourself.
   - **Uses an _Avoid_ term or a name that clashes with the glossary → flag it.**
     Suggest the glossary term.
   - **Fills a gap or changes something the spec left open → fine.** It is the user's
     call. If you see a concrete problem with it (responsibility in the wrong place, an
     invariant that could live in a type but doesn't, a usage that gets awkward), say so
     once, specifically, and then accept their decision.
3. **Hold yourself to the same rules.** Before proposing anything, check it against the
   ledger. Don't slip in a "better" design that quietly reverses a decision.
4. **Revise and re-present.** Update the shape files. In chat, describe what changed and
   why, show updated usage examples for anything whose call sites changed, and list
   what is still open. Stage again (`git add`) so the next diff shows only the user's
   new edits.

Repeat until the user explicitly agrees. Then summarize the agreed shape, component by
component, with the ledger items each one realizes, and ask whether to start the plan.

If shaping exposes a real domain question (one that changes behaviour or meaning, not
just structure), don't settle it in a shape. Name it as a spec gap and ask the user
whether to decide it now or take it back to grilling.

---

## Phase 3: Plan

### Where it goes

If the repo already keeps plans somewhere (e.g. `.claude/plans/<feature>/`), follow
that layout. Otherwise ask the user where to put it. Use one file per build step if the
spec breaks naturally into steps, otherwise a single file.

### If a question comes up while writing

Stop and ask. An open question at this stage means something wasn't settled in shaping.
The plan has no "open questions" section.

### Structure

Write for two readers: the user, who will review and edit it, and an implementing agent
that receives only this document. Use glossary terms throughout.

**Source.** The spec issue (link), the issues it depends on, and the decision ledger
restated in full.

**Reading list.** The docs the implementing agent should read before starting:
glossary, relevant ADRs, linked decision records, the repo's code rules, the spec issue. Say
explicitly that they are reference material and are not to be edited as part of the
implementation.

**Context.** The current state of the code this touches: what exists, where it lives,
how it is structured, and what this spec changes about it. Enough that the implementer
can find their way without having seen the code.

**Agreed shape.** The final shape code, verbatim, per file, with the shape markers
removed. Note which parts are binding (public API, types, ownership) and which are
illustrative.

**Implementation sequence.** The build order and the reason for it: dependencies, what
can be tested early, where the risk is.

**Per-step changes.** For each step: files touched, what the implementation does
(logic, edge cases, error handling, warnings), and which ledger items it realizes.
This is where the behaviour from the spec goes.

**Tests.** Every test the spec asks for, plus any the shape makes necessary. Keep
pinned numbers exactly as the spec states them.

**Success criteria.**
- *Done means*: the tests pass, existing equivalence and round-trip tests still pass,
  and the public API matches the agreed shape.
- *How the code should look*: the structural properties a correct implementation has.
- *How the code should not look*: concrete signs that a decision was broken even though
  tests pass, e.g. "if `Graph` imports `ShapeStore`, L4 was violated".

**Traceability.** A table mapping each ledger entry to the step and test that cover it.
Every entry must appear. If one doesn't, the plan is incomplete.

Present the plan and let the user revise it. Implementation starts only after they sign
off.

### Stash the shapes

After sign-off, take the shape files out of the working tree, so the implementer starts
from the plan instead of patching stubs:

```bash
git stash push --message "shapes: <feature>" -- <every shaped file>
git stash list
```

`git stash push` with paths takes staged and unstaged changes alike. New files are
included because they were staged. Confirm the files are gone from the working tree
(newly created files should no longer exist), then tell the user:

> Shapes stashed as `shapes: <feature>`. View with `git stash show -p stash@{0}`.
> The plan carries the agreed shape verbatim.

If stashing fails, move the files to a scratch directory instead, restore any edited
tracked files with `git restore --staged --worktree`, and tell the user where the copies
are.

---

## Principles

- **Decisions over shapes, shapes over plans.** The spec decides, the shape expresses,
  the plan executes. Never let a later stage quietly change an earlier one.
- **Cite, don't assert.** When you flag or reject something, point at the ledger entry,
  glossary term or code rule behind it.
- **The user is the designer within the decisions.** On anything the spec leaves open,
  give your view once and then follow their call.
- **Show usage, not just declarations.** A signature only shows its problems at the
  call site.
- **Questions first.** If the user asks something, answer it before continuing.
