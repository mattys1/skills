---
name: shape-first-dev
description: >
  A structured, programmer-led feature development workflow that replaces ad-hoc vibe coding
  with deliberate design. Use this skill when a user wants to plan and implement a feature
  collaboratively, especially for non-trivial features where architecture matters. Triggers on
  phrases like "let's plan a feature", "help me design/implement X", "I want to add Y to my
  codebase", "shape-first", "let's spec out", or whenever the user wants to think through a
  feature before jumping into code. The key idea: no implementation planning starts until the
  programmer has shaped every component in code form — even if that code does not compile. Use
  this skill proactively for any feature that involves multiple components or non-obvious coupling.
---

# Shape-First Development

This skill guides a structured feature development process in three distinct phases. The goal is to replace impulsive code generation with deliberate design — keeping the programmer in the driver's seat while letting the agent handle synthesis and implementation heavy-lifting.

**The three phases:**
1. **Specification** — What does this feature need, at a coarse level?
2. **Shaping** — The programmer gives each component a concrete form
3. **Implementation Planning** — Only after all shapes are agreed to

These phases are gates, not suggestions. Do not proceed to phase 3 until phase 2 is complete for every component. Do not proceed to phase 2 until phase 1 has produced a component list the programmer is happy with.

---

## Phase 1: Specification

Your role in this phase is to be a thinking partner, not a decider. The programmer knows their codebase; you help them think out loud.

### How it works

Propose a breakdown of the feature into **components** — units that have a clear identity, a responsibility, and a relationship to other components. A component might be a service, a module, a model, a handler, a hook, whatever fits the codebase's language.

For each component, surface:
- What it does (its responsibility)
- What it exposes (rough public interface, even if vague)
- How it couples to other components (what it calls, what it receives)

**Propose — don't declare.** Write it as a proposal the programmer can react to. Something like:

> "Here's how I'm thinking about this. Tell me where this doesn't match how you see it."

Then iterate. The programmer might collapse two components into one, split one into two, rename things, or change coupling directions. That's the process. Keep refining until the programmer says they're happy with the structure.

**Raise concerns before proceeding.** If you notice something in the proposed component structure that seems off — a component that's trying to do too much, a coupling direction that will cause friction, something that conflicts with patterns you can see in the existing codebase, or a split that doesn't match how the language/ecosystem tends to handle this kind of problem — say so clearly before wrapping up Phase 1. One concern at a time, explained concisely. The programmer may overrule you, and that's their call; your job is to make sure they've heard it. Once they've acknowledged the concern and decided how to proceed, move on without revisiting it.

### What to produce

At the end of Phase 1, you should have a list like:

```
Components:
- UserSessionStore — holds session state, exposes get/set/clear, owned by the auth layer
- LoginHandler — orchestrates login flow, calls SessionStore and AuthService
- AuthService — talks to backend, returns tokens or errors
```

Coupling should be legible: who depends on whom, in what direction.

When the programmer signals they're satisfied, summarize the agreed component list clearly and move to Phase 2. Make it explicit: "Okay, we've agreed on these N components. Now we move to shaping."

---

## Phase 2: Shaping

This is the programmer's phase. Your role here is to wait, review, and ask questions — not to build.

### What a shape is

A shape is a file (or files) where the programmer defines:
- Data models and types
- Public API signatures (function names, parameters, return types — even if stub bodies)
- Parts of the private API they want you to pay attention to
- Comments that convey intent, uncertainty, or constraints
- Questions embedded in the code as comments

Shapes do not need to compile. They do not need to be complete internally. They need to convey a clear design intention. Think of a shape like an architectural sketch — it shows where the walls go, not the plumbing.

**Example of a valid shape (TypeScript):**

```typescript
// UserSessionStore.ts — SHAPE ONLY, not final

// Question: should this be a class or a plain module? Leaning module.
export interface Session {
  userId: string;
  token: string;
  expiresAt: Date;
}

// This is the public API. Keep it minimal.
export function getSession(): Session | null { ... }
export function setSession(session: Session): void { ... }
export function clearSession(): void { ... }

// Not sure if we need this yet — depends on whether LoginHandler needs to check expiry
// export function isExpired(): boolean { ... }
```

### Waiting for shapes

After the programmer acknowledges Phase 2 has started, your stance is: **wait**. Do not offer to shape components yourself. Do not propose implementations. If the programmer shares a shape, review it. If they ask you a question, answer it — their questions take priority over anything else.

**After each shape is shared, run `git diff` (and `git status` for untracked files) to get the full picture of what changed.** Don't rely solely on what the programmer pastes into the conversation — they may have edited multiple files, made incidental fixes, or forgotten to mention a related change. The diff is the ground truth.

```bash
git diff          # changes to tracked files
git status        # catch any untracked new files
git diff --cached # any staged changes
```

If the diff reveals files or changes the programmer didn't mention, surface them: "I also see changes in `X` — is that part of this shape or something else?" Let the programmer clarify before accepting the shape.

When they share a shape, ask yourself honestly:
- Do I understand this component's responsibility?
- Is the public API clear enough that I could implement it without guessing?
- Are there ambiguities that would force me to make architectural decisions they should make?
- Does anything about this shape concern me — an API that seems to put responsibility in the wrong place, a type that's missing something that will obviously be needed, a design choice that looks like it'll cause pain during integration?

**If you have concerns, raise them before saying the shape is accepted.** Be specific: not "this seems off" but "this function takes ownership of X, which means the caller can't Y — is that intentional?" The programmer may say yes, that's intentional, and then the shape is accepted as-is. They have the final say. But do not silently accept a shape that has a real problem in it and then surface that problem during implementation — that undermines the whole point of Phase 2.

Once all concerns are resolved (either addressed in the shape or overruled), confirm the shape is accepted and note what you understand it to mean.

### Tracking shape coverage

Keep mental track of which components from Phase 1 have been shaped. If the programmer submits shapes for some but not all:

- Do NOT proceed to implementation planning
- Ask why the remaining components haven't been shaped yet — are there open questions? Design uncertainty? A dependency on something external? 
- This isn't pushback — it's genuine inquiry. Sometimes an unshapped component is a signal of a real problem. Surface it.

### Delegating shape creation

The programmer may explicitly ask you to shape a component for them. That's fine — do it, following the same conventions (types, stubs, comments for decisions). But do not offer to do this unprompted. The point of shaping is that the programmer makes the architectural decisions; delegation is the exception, not a shortcut.

### End of Phase 2

When every component from Phase 1 has a shape, and you've reviewed each one and confirmed it's workable, summarize:

> "All N components are shaped and I can work with them. Here's what I understand about each: [brief summary]. Ready to draft the implementation plan?"

Wait for the programmer to confirm before proceeding.

---

## Phase 3: Implementation Planning

Only now do you produce a detailed implementation plan. You have everything you need: the agreed structure (Phase 1) and the agreed shapes (Phase 2). This document has two audiences: the programmer, who will review and modify it, and a future implementing agent who will receive it with no other context. Write it for both.

### If an open question surfaces while writing the plan

Stop writing. Do not finish the section. Do not park the question in a note and keep going. An unresolved question at this stage means something wasn't fully settled in Phase 1 or 2 — a component's responsibility was underspecified, a shape left something ambiguous that wasn't caught, or a coupling assumption was never confirmed. Ask the programmer directly. Once it's resolved, continue from where you stopped.

The implementation plan has no "Open Questions" section. If you find yourself wanting to write one, that's the signal to stop and ask.

### Document structure

The plan is a markdown document. It must follow this structure, in order:

---

**Context**

What is the current state of the codebase as it relates to this feature? Describe what exists, how it's structured, and where the new feature fits in or departs from it. If this is a refactor, explain *why* — what problem the current structure has, what the new structure solves, and why the tradeoff is worth it. An implementing agent reading this document should understand the before-state well enough to navigate the codebase without having seen it.

**Component Overview**

A concise restatement of the agreed components from Phase 1, updated to reflect anything that evolved during shaping. One or two sentences per component: what it is, what it's responsible for, how it relates to the others.

**Implementation Sequence**

What gets built in what order, and why. Justify the order — dependency order, what can be tested in isolation first, what carries the most risk and should be settled early. This isn't just a numbered list; it's a rationale.

**Per-Component Plan**

For each component, a detailed implementation description:
- The agreed shape, restated (types, public API signatures)
- What the implementation involves: internal state, logic, edge cases, error handling
- Any decisions that are still open at this point, flagged explicitly
- Integration surface: what it calls, what calls it, what it assumes about those interactions

**Integration**

Where and how the components get wired together. What the call sites look like. If there's an initialization sequence, describe it. If there's shared state, describe who owns it.

**Success Criteria**

This section tells the implementing agent — and the programmer reviewing later — what "done" looks like. It must include:

- *Tests*: what should pass, what test coverage makes sense given the feature, any specific scenarios that must be covered
- *How the code should look*: the structural and stylistic properties of a correct implementation. What patterns should be visible, what invariants should hold. For refactors in particular: if the shapes enforce a specific structure, say so here. "The public API of X should match the shape exactly. Internal implementation detail Y should not be visible to callers."
- *How the code should not look*: especially important for refactors. What patterns, structures, or coupling relationships signal that the implementation went wrong — even if the tests pass. If you're reshaping responsibility boundaries, describe what it looks like when those boundaries are violated. Be concrete: "If Z is being called directly from A, that's a sign the refactor didn't land correctly."

---

The programmer reviews this document. They may modify it, add constraints, or ask you to revise sections. Only when they sign off does implementation begin (by them, or by a delegated agent).

### Stashing shapes before handoff

Once the programmer has signed off on the plan, **stash the shape files before any implementation begins.**

The shape files did their job — they drove the design. Leaving them in the working tree risks the implementing agent patching or anchoring to stub bodies instead of writing clean implementations. A blank slate produces better output.

**How to do it:**

1. Collect the paths of every file that was written or modified as part of shaping in Phase 2.
2. Run:
   ```bash
   git stash push --message "shapes: <feature-name>" -- <file1> <file2> ...
   ```
3. Confirm the stash was created (`git stash list`) and the files are gone from the working tree.
4. Tell the programmer: "Shape files stashed (`git stash list` to verify). The implementing agent will work from a clean slate."

If the programmer or implementing agent ever needs to reference a shape, they can inspect it with `git stash show -p stash@{0}` without popping it. The shapes are preserved — just out of the way.

**If `git stash` fails** (untracked files, no git repo, etc.), fall back to moving the shape files to a temporary directory (e.g. `/tmp/shapes-<feature-name>/`) and note the location. Don't block on this — the implementing agent's clean slate matters more than the stash mechanism.

---

## General Principles

**Programmer primacy.** The programmer decides architecture. You synthesize, propose, and implement — but never override. If you disagree with a design decision, say so once, clearly, and then respect their call.

**Explicit transitions.** Always make phase transitions explicit. Don't drift from spec into shape or shape into planning. Name the phases and say when you're moving.

**Questions first.** If the programmer asks you something at any point, answer it before continuing with whatever the current phase activity is.

**Don't paper over uncertainty.** If something is unclear — in a shape, in the spec, anywhere — say so. It's much cheaper to resolve ambiguity in Phase 2 than to undo wrong implementation in Phase 3.
