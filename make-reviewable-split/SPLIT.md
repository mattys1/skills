# Splitter

You build the **review branch** `<source branch>-review`: the source branch's full change, rebuilt from the base branch as a series of **commits** a human reviews one at a time. You read the source branch and never modify it.

## Groups

A **group** is one coherent change a reviewer reads as a unit, ending at a state a reviewer could stop at. The review branch is a sequence of groups.

Groups are cut from the **final diff** (`git diff <base>...<source>`). The source branch's history is a hint for grouping, never a state to replay: code it wrote in one ticket and rewrote in a later one reaches the review branch only in its final form.

- **Ticket groups**: when the source branch's first-parent history (`git log --first-parent --reverse <base>..<source>`) is a chain of merge commits (one per ticket, as `implement-spec` produces), each ticket is one group, in merge order. `git blame --first-parent <base>..<source> -- <file>` attributes each final line to the merge that last wrote it, which places it in that ticket's group. Keep a unit (function, type, test block) whole in the group that owns most of it, so a group may absorb lines blamed to another ticket; drop a ticket whose lines were all absorbed or rewritten.
- **Planned groups**: otherwise, plan them from the final diff, using the source PR's commits and linked issues as hints.

## Final form

Every line of the final diff is written by exactly one commit, as it stands in the source branch's tree; later commits add around it and leave it alone. Order commits by dependency (definitions before their callers) so each one builds from final-form lines alone. When two steps only build through an intermediate version of some line, merge them into one commit.

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

`<group>` is the group's ticket (`#227`) for ticket groups, or a short slug for planned groups. Every other commit is titled `<summary> (<group>)`. **Prod** commits (1–4) carry zero test code, so filtering out `test(` commits leaves exactly the production change.

Tests often live in the same file as the code (Rust `#[cfg(test)] mod tests { … }`). The prod commit writes the file without the test block; the test commit adds the block. A declaration that pulls in a separate test file (`#[cfg(test)] mod tests;`) goes in the test commit with the file it declares.

A group under ~500 reviewable lines can be one prod commit, still followed by its test and data commits.

## Procedure

Work in a fresh worktree of the base branch on a new branch `<source branch>-review`.

1. Determine the groups from the final diff, assigning every changed hunk to one group.
2. For each group: plan its commits, then build them by writing each commit's file states from the source branch (`git checkout <source> -- <path>` for a file that lands whole; edit the file for one that lands in parts, copying final-form lines). Commit each step.
3. **Check** after each group: every commit in the group builds with the repo's fast build check (for Rust: `git rebase -x 'cargo check --all-targets' <group start>`, which rewrites nothing when every check passes).
4. **Check** at the end, all must hold:
   - `git diff <source branch> HEAD` is empty;
   - the full test suite passes at HEAD;
   - the branch is in final form: per file, the added and deleted counts summed over `git log --numstat --format= <base>..HEAD` equal those in `git diff --numstat <base> HEAD`. A file whose sums exceed its diff has churn: reshape the commits that touch it.
5. If a check fails and reshaping the commits doesn't fix it, delete the review branch and report failure with the reason.
6. Report the series: per group, each commit's short hash, title, and reviewable lines.
