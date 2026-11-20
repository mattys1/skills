---
name: implement-spec-stacked
description: "Implement the result of /to-spec and /to-tickets in code, delivered as a stack of per-ticket PRs."
disable-model-invocation: true
---

You have been provided a spec. This spec should have tickets associated with it, describing how to implement the spec.

The issue tracker should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`.

The goal is the entire spec implemented as a **stack**: a chain of PRs, one per ticket, each based on the branch of the PR below it, the bottom one based on main. Each PR shows exactly one ticket's change, and resolves that ticket. The stack merges into main bottom-up, and together the PRs resolve the spec.

The tickets are not a list of steps. They are a **task graph** with blocking relationships between them. This means there is always a **frontier** of tickets which are ready to be grabbed. Tickets are built concurrently, but they join the stack one at a time, in the order they finish: the stack is the task graph linearised by completion order.

The **stack top** is the branch of the topmost PR in the stack (main, before the first ticket joins).

Start every subagent with a fresh context window.

Communication to and from subagents should be sparse. Communicate primarily through **context pointers**: to the spec, tickets, research notes, and previous commits. Don't duplicate information already available via pointers.

**Implementer subagents** should be run in the background where possible for maximum concurrency.

## Steps

1. Read the spec and tickets to understand the task graph.

2. (optional) Use an **exploration subagent** to conduct any exploration required by the tickets - relevant codebase files or external documentation. Ensure the exploration subagent can save files - it should save its markdown notes in a directory outside the repo, accessible by all future subagents. This lets **implementer subagents** focus on implementation rather than exploration.

3. Use a fresh **implementer subagent** for each ticket, each in its own worktree on its own branch. Each implementer subagent:
   - confirms its worktree is based on the current stack top before starting, and resets onto it if not;
   - calls the Skill tool with `tdd` to build the ticket.

   The stack top may move while it works; the stacker catches its branch up.

4. Once an **implementer subagent** completes, add its branch to the stack with a **stacker subagent**. Stack one branch at a time; the stacker:
   - confirms the branch contains the current stack top (`git merge-base --is-ancestor <stack top> <branch>`), merging the stack top in if not;
   - pushes the branch and opens its PR with the Skill tool `pr`, base set to the current stack top, marked as closing its ticket, with its position in the stack (`Stack: 3/7, on #<PR below>`) in the body.

   The new branch becomes the stack top.

5. If this changes the **frontier** of available tickets, kick off more **implementer subagents** to work on the new tickets, based on the new stack top. This allows for maximum concurrency.

6. Once all tickets are stacked, call the Skill tool with `code-review` on the stack top against main. Fix all issues raised by the code review in a single fresh **implementer subagent**, and stack its branch as one more PR (`Review fixes for <spec>`), marked as closing the spec.

7. Edit the bottom PR's description to list the whole stack in order, so the reviewer has one entry point. Report the stack to the user.

8. **Review feedback**, whenever the user relays comments on a PR in the stack: a fresh **implementer subagent** adds fix commits to that PR's branch, then **restacks** upward: merges each branch into the one above it, bottom to top, re-running the tests at each, and pushes every branch it touched. Restacking only adds merge commits; nothing is rebased or force-pushed, so review comments stay anchored.

9. **Landing**: merge the stack bottom-up, each PR with a merge commit (squash or rebase merges break the PRs above). After each merge, retarget the next PR's base to main if GitHub didn't, and delete the merged branch.

10. Clean up all **implementer subagent** worktrees.
