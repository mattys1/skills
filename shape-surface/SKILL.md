---
name: shape-surface
disable-model-invocation: true
description: >
  Light shape-plan: agree with the user on a spec's public surface and seams, then post
  it to the spec as a shape note. Usage: /shape-surface <spec issue>
---

# Shape-surface

You take a spec whose decisions are already **locked** and agree with the user on its
**surface**: what the rest of the codebase sees of it, and which ticket brings each part.
Everything behind the surface is the implementer's call. The result is a **shape note**,
short enough to post as a comment on the spec, where implementers read it next to their
tickets.

Four phases; say out loud when you move from one to the next:

0. **Gather**: load the spec, its tickets and the code at its boundary.
1. **Propose**: draft the shape note.
2. **Iterate**: read the user's edits, revise until they agree.
3. **Publish**: post the note to the spec and point each ticket at it.

---

## The surface

An item is on the surface when code outside its module names it: a caller, another
ticket's code, a test, a binary. Stop at the module boundary. Private types, fields,
helpers and algorithms belong to the implementer, unless the spec pins one down and
another module depends on it.

A shape note holds:

- **Items**: each new, changed or removed surface item as one signature line, with no body,
  tagged with the ticket that introduces it. Add a one-line contract only where the
  signature leaves out something a caller relies on, citing the spec part it comes from:
  ```
  - `Database::open(dir: &Path) -> Result<Database, OpenError>` (#88). Fails while
    another process holds the directory lock. (Behaviour 2)
  ```
- **Seams**: who owns what and who calls whom across modules, including what a module
  must stay unaware of.
- **Moves**: callers that change because an item went away or changed.
- **Guardrails**: signs a decision was broken that a green suite won't catch, each citing
  the spec: "`Graph` imports `ShapeStore` → Implementation Decisions, Store".
- **Settled**: gaps answered during shaping, one line each.

The ticket tag carries the multi-ticket split. When parallel tickets share an item, the
introducing ticket owns it and the others code against the note's signature, so
implementers working side by side don't each invent their own version.

Use the codebase's naming, error types and idioms, and name every domain concept with its
`GLOSSARY.md` term.

**Budget:** the note fits on one screen, about 60 lines. A note growing past that has
leaked behind the boundary; cut private items first.

---

## Sources of truth

Resolve disagreements in this order, highest first: the spec's resolution (body and
comments); changes agreed on its parent issue; linked specs it builds on; `GLOSSARY.md`,
`docs/adr/` and decision records the spec links; the code as it is today, which is the
before-state.

The spec is the ledger: cite its numbered Behaviours and its Implementation Decision
headings directly. The note expresses locked decisions and never revisits them. Domain
docs and issues are changed only in a **grilling** session the user runs. When shaping
hits a contradiction with a locked decision, a missing glossary term, or a question that
changes behaviour rather than structure, name it, tell the user it belongs to grilling,
and carry on with what is settled.

---

## Phase 0: Gather

Read the spec with its comments, every ticket with its blocking relations, and the code
rules in `CLAUDE.md`. Check:

- **No resolution, or decisions still open:** stop. This skill needs locked decisions.
- **A prerequisite spec hasn't landed in the code:** say so; the surface builds on that
  spec's agreed design, not today's code.

Code reaches you through a **boundary map**. Spawn one fresh, non-fork subagent on haiku
with `effort: "high"`. It starts without this session's context, so give it the repo path,
the spec link to fetch itself, and the modules the spec touches. It returns, with a
`path:line` on every entry:

- the **public items** of those modules: types, signatures, error types, settings,
  test-support exports;
- the **call sites** across module boundaries: who calls them from where, binaries and
  tests included;
- the **conventions** the surface must follow: naming, error types, module layout.

Open a `path:line` only where an item or a seam needs the body.

Then list the **gaps**: surface the spec leaves open, such as a return or error type,
which module owns an item, or which ticket introduces a shared one.

---

## Phase 1: Propose

Write the draft note to `/tmp/shape-note-<repo>-<spec number>.md`, with each gap as a
`Q1`, `Q2`, … line at the end. The user reads these cold, so each one stands alone: what
is being decided, the spec part or ticket it comes from, and your proposal.

Snapshot the draft (`cp <note> <note>.base`) so the next diff shows only the user's edits.

In chat: the note, then one usage snippet for each new entry point (a constructor, a
top-level call), showing it at a real call site from the boundary map. Tell the user:

> Shape note round 1 is in `<note>`. Edit it directly or reply in chat. When you're done,
> tell me and I'll read your edits.

---

## Phase 2: Iterate

1. **Read the edits** with `diff -u <note>.base <note>`, plus anything said in chat.
2. **Check each edit:**
   - **Contradicts the spec → flag it**, citing the part, and route it to grilling.
   - **Clashes with the glossary → flag it** and suggest the glossary term.
   - **Fills a gap → the user's call.** If you see a concrete problem (an item in the wrong
     module, an awkward call site, a ticket using an item before the ticket that
     introduces it lands), say so once, specifically, then follow their decision.
3. **Revise** the note: move answered `Q`s into Settled, and re-snapshot. In chat: what
   changed and why, and the `Q`s still open.

Repeat until the user explicitly agrees.

---

## Phase 3: Publish

Before presenting the final note, check coverage:

- every item the spec names that code outside its module will use appears in Items;
- every ticket appears in at least one tag, or the note says it changes no surface;
- no `Q` is left.

Show the final note. After sign-off, post it as a comment on the spec, headed
`## Shape note` and opening with: "Binding on the surface below; everything behind it is
the implementer's call." Then append to the body of each ticket that introduces or uses a
surface item, where its implementer is sure to read it:

```
## Shape note

<comment link>. Code against its signatures; if one can't hold, report it instead of
changing it.
```

Report the comment link and the next command: `/implement-spec <spec>`.
