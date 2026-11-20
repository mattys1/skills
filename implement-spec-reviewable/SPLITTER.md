# Splitter

You build the **review branch**: the integration branch's full change, rebuilt from main as a series of **commits** a human reviews one at a time. You read the integration branch and never modify it.

## Groups

The integration branch's first-parent history is a chain of merges, one per ticket plus the review fixes:

```sh
git log --first-parent --reverse --format='%H %s' main..<integration branch>
```

Each merge is one **group**. On the review branch, group _k_ ends exactly at merge _k_'s tree; its commits rebuild the step from merge _k−1_'s tree (main's, for the first) to merge _k_'s. The groups' order is the merges' order.

## Line counts

A commit's **reviewable lines** are its added plus deleted lines, excluding test code and bulk data (lockfiles, generated code, measurement CSVs, recorded outputs). **Test harness** code (shared test support, oracles, fuzzers, workload generators, bench frameworks) counts as reviewable.

Aim for about **500** reviewable lines per commit, with **1000** as the cap. The cap yields to logic: a unit that only makes sense whole stays whole, over the cap, and its commit message body says why in one line.

## Shaping a group

Each commit is one **logical step** a reviewer can understand alone. Order a group's commits:

1. preparatory refactors and renames (behaviour unchanged);
2. new types, data structures, core logic, and the test harness they need;
3. wiring into existing callers;
4. tools on top: GUI, CLI, benches;
5. **test commits**: all of the group's test code, titled `test(#<ticket>): …`;
6. bulk data, titled `data(#<ticket>): …`.

Every other commit is titled `<summary> (#<ticket>)`. **Prod** commits (1–4) carry zero test code, so filtering out `test(` commits leaves exactly the production change.

Tests often live in the same file as the code (Rust `#[cfg(test)] mod tests { … }`). The prod commit writes the file without the test block; the test commit adds the block. A declaration that pulls in a separate test file (`#[cfg(test)] mod tests;`) goes in the test commit with the file it declares.

A group under ~500 reviewable lines can be one prod commit, still followed by its test and data commits.

## Procedure

Work in a fresh worktree of main on a new branch `<integration branch>-review`.

1. List the groups.
2. For each group: read `git diff <previous merge> <merge>`, plan its commits, then build them by writing each commit's file states directly (`git checkout <merge> -- <path>` for a file that lands whole; edit the file for one that lands in parts). Commit each step.
3. **Check** after each group, all must hold:
   - `git diff <merge> HEAD` is empty;
   - every commit in the group builds with the repo's fast build check (for Rust: `git rebase -x 'cargo check --all-targets' <group start>`, which rewrites nothing when every check passes).
4. At the end, the full test suite passes at HEAD, and `git diff <integration branch> HEAD` is empty.
5. If a check fails and reshaping the group's commits doesn't fix it, delete the review branch and report failure with the reason.
6. Report the series: per group, each commit's short hash, title, and reviewable lines.
