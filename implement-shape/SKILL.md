---
name: implement-shape
disable-model-invocation: true
description: >
  Implement a committed shape from its plan dir. Usage: /implement-shape <plan dir>
---

# Implement-shape

The argument is a **plan dir** written by `shape-plan`; if none was given, ask for it.

The committed **skeleton** is binding: its types, signatures, visibility and ownership stay
as committed. The plan is a delta over it: the ledger, the steps with their notes, the
guardrails. Skeleton and plan together are the whole spec; the work is filling in every
`todo!()`.

## 1. Load

1. Read every file in the plan dir.
2. Find the **shape commit**, the one that added the plan:
   `git log --diff-filter=A --format=%h -- <plan dir>/plan.md`.
3. Read every file it touched (`git show --stat <shape commit>`).

## 2. Implement, step by step

For each step of the plan:

1. Fill in the shaped items the step names and implement its test stubs, applying every
   note: invariants, internal behaviour, corrections, gotchas.
2. Run the step's tests, then the full suite. The step is done when its stubs are green and
   the existing suite stays green.
3. Commit it to the current branch: `<feature>: <step title>`.

Work within the skeleton. When it can't hold (a signature can't carry what a note
requires, or the code contradicts a ledger entry), stop and show the user the conflict,
citing the entry. The skeleton changes only with their agreement.

## 3. Finish

Strip the ledger tags (`// L3`) from the code, keeping the comments' prose. Done when:

- `grep` finds no `todo!()` or ledger tag in the files the shape commit touched;
- the full suite and the repo's lints are green;
- every guardrail in the plan is checked against `git diff <shape commit>`, and none is
  tripped.

Report the step commits, any skeleton change the user agreed to, and the guardrail check.
