---
name: make-reviewable-split
description: "Republish a PR as one PR of small, grouped commits a human reviews one at a time. Usage: /make-reviewable-split <PR>"
disable-model-invocation: true
---

You have been given a **source PR**. Its head branch is the **source branch**, its base the **base branch**. The goal is the source PR's change republished as one new PR from the **review branch** `<source branch>-review`: the same final tree, rebuilt from the base branch as small logical commits in **groups**, with test code in separate `test(...)` commits. The source PR is closed in favour of it.

The source branch is frozen from step 1 on: nothing is committed to it again.

## Steps

1. Read the source PR: title, body, linked issues, commits, diff size.

2. Build the review branch with a fresh **splitter subagent** (`model: "haiku"`, `effort: "high"`), pointing it at [`SPLIT.md`](SPLIT.md) (by absolute path), the source PR, the source branch, and the base branch. If it reports failure, run a fresh splitter with `model: "sonnet"`, `effort: "medium"`. If that fails too, stop and report to the user.

3. Push the review branch and open its PR with the Skill tool `pr`, linking the source PR and carrying the closing keywords for every issue the source PR resolves. The body includes the **review guide**: one section per group, listing its commits as the splitter reported them, with `test(...)` commits marked skippable.

4. Comment on the source PR, linking the new PR, then close it.

5. Report the new PR to the user.

## After publishing

**Review feedback**, whenever the user relays comments: a fresh **implementer subagent** adds fix commits on top of the review branch in a worktree, titled `fix(review): …` (test-only fixes `test(review): …`), and pushes. The review branch is never rebased or force-pushed, so review comments stay anchored.

**Landing**: merge the PR, then delete the source branch, the review branch, and any worktrees.
