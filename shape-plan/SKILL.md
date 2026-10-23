---
name: shape-plan
disable-model-invocation: true
description: >
  Shape a resolved spec's code with the user (types, signatures, seams written
  into the real files), then hand over a compiling skeleton and a plan.
---

# Shape-plan

You take a spec whose decisions are already **locked** and agree with the user on the
**shape** of its code: types, signatures and seams, written as illustrative code straight
into the files where it will live. The user edits it in place; you read their edits back
with `git diff`. The agreed shape then becomes the implementer's skeleton, and the plan
carries only what the shape can't say.

Four phases; say out loud when you move from one to the next:

0. **Gather**: load the spec and everything it rests on, then build the decision ledger.
1. **Propose**: write the shape into the code and explain it in chat, with usage examples.
2. **Iterate**: read the user's edits, check them against the ledger, revise until they agree.
3. **Hand over**: finish the shape into a compiling skeleton, write the plan as a **delta**
   over it, and commit both.

---

## Sources of truth

Resolve disagreements in this order, highest first:

1. **The spec's resolution.** What the spec / issue has resolved (body or comments), plus
   any "Requirement from …" sections other specs pushed into it.
2. **Changes agreed on the parent issue / map** (e.g. "Changes agreed while charting").
   They override older decision records they explicitly reverse.
3. **Linked spec issues** this one builds on (e.g. the Shape store spec builds on the
   compression spec's `Imported` map).
4. **Domain docs:** `GLOSSARY.md` (or `GLOSSARY-MAP.md`), which governs naming;
   `docs/adr/`; and any decision record the spec or its parent explicitly links to.
5. **The code as it is today**: the before-state, not what should be built.
6. **Older material** (a thesis, old notes): background only.

The shape expresses locked decisions; it never revisits them. This binds your proposals
as much as the user's edits.

### Grilling owns the decisions

Read the domain docs as `docs/agents/domain.md` describes, if the repo has it. The docs and
issues are read-only here: `GLOSSARY.md`, ADRs, linked decision records and issues are
changed only in a **grilling** session the user runs. Whenever shaping hits one of these,
name it, tell the user it belongs to grilling, and carry on with what is settled:

- a proposed shape (yours or theirs) contradicts a locked decision;
- a missing glossary term or an ADR conflict;
- a domain question: one that changes behaviour or meaning, not just structure. Ask
  whether to decide it now or take it to grilling.

---

## Phase 0: Gather

Before doing anything, create a new branch, if currently on `master` or `main`.

Read the sources above, including issue comments, the code the spec touches, and the code
rules in `CLAUDE.md`; the code rules constrain the shape as much as the spec does. Check:

- **No resolution, or decisions still open:** stop. This skill needs locked decisions.
- **A prerequisite spec hasn't landed in the code:** say so; the shape builds on that
  spec's agreed design, not today's code.
- **A file you'll shape has uncommitted changes:** ask the user to commit or stash it
  first, or the shape commit would take their changes with it.

### The decision ledger

Distill the locked decisions that bear on the code into a numbered **ledger**, `L1`, `L2`, …,
one decision per entry, quoted or closely paraphrased, with its source:

```
L1  EdgeData gains `shape: Nullable<ShapeID>`, mirroring `properties`. (#4 Reference from Edge)
L2  Only interior Shape points are stored; Node coordinates are not repeated. (#4 What is stored)
L3  Shape points are stored in from→to travel order; reversed for Reverse ways. (#4 Point order, #3 Direction)
L4  Graph doesn't know about the ShapeStore; Database owns it. (#4 Store)
...
```

Include behavioural decisions too, marking the ones that pin down shape. Then list
**gaps**: structural things the shape needs that the spec leaves open. Gaps are the
user's to fill during shaping.

Show ledger and gaps in chat. The numbering stays stable for the session; every flag you
raise cites an entry.

---

## Phase 1: Propose

### What goes into a shape

Illustrative code; it need not compile yet:

- data types: structs, enums, newtypes, with field types;
- public API: signatures with `todo!()` (or the language's equivalent) bodies;
- trait impls, derives and visibility that matter to the design;
- the seams: who owns what, who calls whom, what crosses the boundary;
- private details only where the spec pins them down (a field layout, a const parameter);
- short doc comments stating intent, with ledger tags (`// L3`);
- open points as `// Q:` comments.

Algorithms and logic stay out. Follow the codebase's naming, module layout, error types and
idioms, and name every domain concept with its glossary term (never one under _Avoid_).

### Where it goes

Where it will really live: edit the existing file in place, or create the new file at its
intended path. In existing files, show only what is new or changed. Mark every shaped
region:

```rust
// SHAPE(shape-store) ─────────────────────────────
pub(in crate::database) struct ShapeStore {
    pool: ClusterPool<ShapePoint, 4, ShapesFile>, // L5
}
// ───────────────────────────────── /SHAPE
```

### What goes into chat

Per component:

1. **Summary**: what it is, what it owns, which ledger entries it realizes.
2. **Key code**, or a pointer (`src/database/shape.rs:12`) if long.
3. **Usage**: two to four realistic snippets from its real call sites (the importer
   filling it, the exporter reading it, a test asserting on it). A signature shows its
   problems only at the call site.

Then list the `// Q:` points and gaps for the user to decide.

### End the round

Stage the shape files with `git add` (`-N` is not enough for new files). The index now
holds your proposal, so the user's edits show up as unstaged changes. Tell the user:

> Shape round 1 is in (staged). Edit the files directly or reply in chat. When you're
> done, tell me and I'll read your edits with `git diff`.

---

## Phase 2: Iterate

When the user has edited or replied:

1. **Read the edits**: `git diff`, plus `git status` for new untracked files. If the diff
   touches files you didn't shape, ask whether they are part of the shape.
2. **Check every change against the ledger**, citing the entry or glossary term behind
   each flag:
   - **Contradicts a locked decision → reject it**, in one or two sentences:
     > Rejecting the `shape: Vec<ShapePoint>` field in `EdgeData`: L1 says the Edge
     > holds `Nullable<ShapeID>` into a separate store. If you want to reopen that,
     > it's a grilling question for #4, not a shape change.

     Leave the user's text in place, but build later rounds and the plan on the ledger.
   - **Clashes with the glossary → flag it** and suggest the glossary term.
   - **Fills a gap → the user's call.** If you see a concrete problem (responsibility in
     the wrong place, an invariant that could live in a type, an awkward call site), say
     so once, specifically, then follow their decision.
3. **Revise.** Update the shape files. In chat: what changed and why, updated usage for
   anything whose call sites changed, what is still open. Stage again, so the next diff
   shows only the user's new edits.

Repeat until the user explicitly agrees. Then summarize the agreed shape per component,
with the ledger entries each realizes, and ask whether to start the hand-over.

---

## Phase 3: Hand over

The agreed shape stays in the code as the implementer's skeleton. The plan is a **delta**
over it: only what the implementer can't read off the shape or the code.

### Finish the shape

1. **Contracts into doc comments.** Behaviour a caller relies on (what a function
   guarantees, rejects, or drops silently) goes in the doc comment of its item, where it
   outlives the plan.
2. **Tests as stubs.** Every test the spec asks for, plus any the shape makes necessary,
   at its real location: the name, a `todo!()` body, and a comment stating what it
   asserts, with pinned numbers exactly as the spec states them. Stubs stay red until
   their step implements them; that is the progress signal.
3. **Build.** The skeleton compiles (`cargo check` or the equivalent), so the existing
   suite runs against it from the first step.

Leave the shaping comments (`SHAPE` markers, resolved `// Q:` points) in place: a
subagent strips them before the commit. Done when it builds.

### Write the plan

The plan is a **plan dir** of its own, `<plans>/<feature>/`, holding `plan.md` (plus one file
per step if the steps are long enough to split). `<plans>` is the repo's plan location if it
has one (e.g. `.claude/plans/`); otherwise ask where. A question that comes up while writing
means shaping left something open: stop and ask.

Every line passes the **delta** test: could the implementer read this off the shape or the
code? If so, cut it. Readers: the user, who reviews it, and an implementing agent that gets
this file plus the committed skeleton.

- **Header.** The spec issue link, and one line: the shape in the code is binding; its
  `// L…` tags cite the ledger below.
- **Ledger.** The ledger and the gaps settled during shaping, numbered as in the session,
  in full: the code cites it.
- **Steps.** The build order and why. Per step, the shaped items it fills (by path), then
  notes of four kinds only:
  - **invariants** the types don't enforce: an order, a uniqueness, "exactly one",
    all-or-nothing;
  - **internal behaviour** no doc comment states: edge cases, error paths, which warning
    fires;
  - **corrections** to code outside the shape: old behaviour to remove, call sites to
    update, existing tests whose pinned values change;
  - **gotchas** found while shaping: a pattern to follow, a helper to avoid.
- **Guardrails.** Signs a decision was broken that a green suite won't catch, each citing
  its entry: "`Graph` imports `ShapeStore` → L4".
- **Done.** Every stub implemented and green, the existing suite green, no `todo!()` or
  ledger tag left.

Before presenting, check coverage: every ledger entry is realized by a tag in the shape, a
test stub, or a plan note. An entry with none is a hole in the hand-over. Then let the user
revise the plan and the shape.

### Commit

After sign-off, stage skeleton and plan (`git add <every shaped file> <plan files>`), so
any further change shows up as unstaged.

**Strip the shaping comments.** Spawn one fresh, non-fork subagent (a fork inherits your
model) on a cheap model (a model like haiku, `effort: "medium"`). It starts without this
session's context, so give it the list of shaped files, tell it that `git diff HEAD` shows
what shaping added, and these rules:

- **Remove**: `SHAPE(…)` and `/SHAPE` marker lines; `// Q:` comments; comments addressed
  to the user or recording the shaping discussion (alternatives considered, "agreed in
  round 2", rejected edits).
- **Keep**: doc comments stating intent or contracts; ledger tags (`// L3`); the comment
  in each test stub stating what it asserts; every comment that was in the file before
  shaping.
- Delete comments only; leave all code as it is. Don't stage or commit.

When it returns, check `git diff`: every changed line is a removed comment from the list
above. Restore anything else.

Then commit skeleton and plan together, so `git diff` against this commit shows exactly the
implementation:

```bash
git add <every shaped file> <plan files>
git commit -m "shape: <feature>"
```

If a hook rejects it, show the user the failure and fix the shape. Report the commit hash and
the next command: `/implement-shape <plan dir>`.
