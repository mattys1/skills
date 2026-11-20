---
name: make-reviewable-stack
description: "Republish a PR as a stack of PRs, one per group of its original commits. Usage: /make-reviewable-stack <PR>"
disable-model-invocation: true
---

You have been given a **source PR**. Its head branch is the **source branch**, its base the **base branch**. The goal is the source PR's change republished as a **stack**: one PR per **group**, each based on the branch of the PR below it, the bottom one on the base branch. The source PR is closed in favour of the stack.

Stacked PRs keep the original commits: each group ends at a commit on the source branch's first-parent history (`git log --first-parent --reverse <base>..<source>`), and its branch `<source branch>-stack-<k>` is a pointer to that commit. Nothing is rebuilt or rewritten. The source branch is frozen from step 1 on: nothing is committed to it again.

## Groups

A group is one coherent change a reviewer reads as a unit.

- **Merge groups**: when the first-parent history is a chain of merge commits (one per ticket, as `implement-spec` produces), each merge is one group, in that order.
- **Planned groups**: otherwise, choose cut points between first-parent commits, using the commits' content and the PR's linked issues as hints.

A single commit too big to stand as one group means stacking can't help this PR: report that and recommend `/make-reviewable-split`.

## Steps

1. Read the source PR: title, body, linked issues, commits, diff size. Determine the groups.

2. Create `<source branch>-stack-<k>` at each group's last commit (`git branch <name> <commit>`), and push them.

3. Open one PR per group with the Skill tool `pr`, bottom first: base is the branch below (the base branch for the bottom one), body has `Stack: k/n, on #<PR below>`, links the source PR, and closes the issues its group resolves; the top PR also closes anything else the source PR closed, such as a spec. Then edit the bottom PR's body to list the whole stack in order.

4. Comment on the source PR, linking the stack's bottom PR, then close it.

5. Report the stack to the user.

## After publishing

**Review feedback**, whenever the user relays comments on a PR in the stack: a fresh **implementer subagent** adds fix commits to that PR's branch in a worktree, then **restacks** upward: merges each branch into the one above it, bottom to top, re-running the tests at each, and pushes every branch it touched. Nothing is rebased or force-pushed, so review comments stay anchored.

**Landing**: merge bottom-up, each PR with a merge commit (squash or rebase merges break the PRs above). After each merge, retarget the next PR's base to the base branch if GitHub didn't. Once all have merged, delete the source branch, the stack branches, and any worktrees.
