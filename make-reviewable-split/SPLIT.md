# Splitter

You build the **review branch** `<source branch>-review`: the source branch's full change, rebuilt from the base branch as a series of **commits** a human reviews one at a time. You read the source branch and never modify it.

## Groups

A **group** is one coherent change a reviewer reads as a unit, ending at a state a reviewer could stop at. The review branch is a sequence of groups.

- **Merge groups**: when the source branch's first-parent history (`git log --first-parent --reverse <base>..<source>`) is a chain of merge commits (one per ticket, as `implement-spec` produces), each merge is one group, in that order.
- **Planned groups**: otherwise, plan them from the whole diff, using the source PR's commits and linked issues as hints.

## Line counts

A commit's **reviewable lines** are its added plus deleted lines, excluding test code and bulk data (lockfiles, generated code, measurement CSVs, recorded outputs). **Test harness** code (shared test support, oracles, fuzzers, workload generators, bench frameworks) counts as reviewable.

Aim for about **500** reviewable lines per commit, with **1000** as the cap. The cap yields to logic: a unit that only makes sense whole stays whole, over the cap, and its commit message body says why in one line.

## Shaping a group

Each commit is one **logical step** a reviewer can understand alone. Order a group's commits:

1. preparatory refactors and renames (behaviour unchanged);
2. new types, data structures, core logic, and the test harness they need;
3. wiring into existing callers;
4. tools on top: GUI, CLI, benches;
5. **test commits**: all of the group's test code, titled `test(<group>): …`;
6. bulk data, titled `data(<group>): …`.

`<group>` is the group's ticket (`#227`) for merge groups, or a short slug for planned groups. Every other commit is titled `<summary> (<group>)`. **Prod** commits (1–4) carry zero test code, so filtering out `test(` commits leaves exactly the production change.

Tests often live in the same file as the code (Rust `#[cfg(test)] mod tests { … }`). The prod commit writes the file without the test block; the test commit adds the block. A declaration that pulls in a separate test file (`#[cfg(test)] mod tests;`) goes in the test commit with the file it declares.

A group under ~500 reviewable lines can be one prod commit, still followed by its test and data commits.

## Procedure

Work in a fresh worktree of the base branch on a new branch `<source branch>-review`.

1. Determine the groups. For each, its **target**: a merge group's target is its merge commit's tree; a planned group's target is the state you plan for it, and the last group's target is the source branch's tree.
2. For each group: read the diff from the previous target to this one, plan its commits, then build them by writing each commit's file states directly (`git checkout <commit> -- <path>` for a file that lands whole; edit the file for one that lands in parts). Commit each step.
3. **Check** after each group, all must hold:
   - for a merge group, `git diff <merge> HEAD` is empty;
   - every commit in the group builds with the repo's fast build check (for Rust: `git rebase -x 'cargo check --all-targets' <group start>`, which rewrites nothing when every check passes).
4. At the end, the full test suite passes at HEAD, and `git diff <source branch> HEAD` is empty.
5. If a check fails and reshaping the group's commits doesn't fix it, delete the review branch and report failure with the reason.
6. Report the series: per group, each commit's short hash, title, and reviewable lines.
