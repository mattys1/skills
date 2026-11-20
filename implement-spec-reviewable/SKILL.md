---
name: implement-spec-reviewable
description: "Implement the result of /to-spec and /to-tickets in code, delivered as one PR a human can review commit by commit."
disable-model-invocation: true
---

You have been provided a spec. This spec should have tickets associated with it, describing how to implement the spec.

The issue tracker should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`.

The goal is the entire spec implemented on a single **integration branch**, then presented on a **review branch** (`<integration branch>-review`) as one PR a human reviews commit by commit. The review branch is what gets merged, and its PR resolves the spec and every ticket.

The tickets are not a list of steps. They are a **task graph** with blocking relationships between them. This means there is always a **frontier** of tickets which are ready to be grabbed.

Start every subagent with a fresh context window.

Communication to and from subagents should be sparse. Communicate primarily through **context pointers**: to the spec, tickets, research notes, and previous commits. Don't duplicate information already available via pointers.

**Implementer subagents** should be run in the background where possible for maximum concurrency.

## Steps

1. Read the spec and tickets to understand the task graph.

2. (optional) Use an **exploration subagent** to conduct any exploration required by the tickets - relevant codebase files or external documentation. Ensure the exploration subagent can save files - it should save its markdown notes in a directory outside the repo, accessible by all future subagents. This lets **implementer subagents** focus on implementation rather than exploration.

3. Create the integration branch from main. It gets no PR.

4. Use a fresh **implementer subagent** for each ticket, each in its own worktree on its own branch. Each implementer subagent:
   - confirms its worktree is based on the integration branch before starting, and resets onto it if not;
   - calls the Skill tool with `tdd` to build the ticket;
   - merges the integration branch tip into its own branch before reporting done

5. Once an **implementer subagent** completes, merge its work to the integration branch with a **merger subagent**, using a merge commit (`--no-ff`) titled `Merge <ticket branch> (#<ticket>)`. These first-parent merges are what the splitter later groups the review branch by.

6. If this changes the **frontier** of available tickets, kick off more **implementer subagents** to work on the new tickets. This allows for maximum concurrency.

7. Once all tickets are complete, call the Skill tool with `code-review` on the integration branch. Fix all issues raised by the code review in a single fresh **implementer subagent**, and merge it like a ticket (titled `Merge <branch> (review fixes)`).

8. Build the review branch with a fresh **splitter subagent** (`model: "haiku"`, `effort: "high"`), pointing it at [`SPLITTER.md`](SPLITTER.md) (by absolute path), the integration branch, and main. If it reports failure, run a fresh splitter with `model: "sonnet"`, `effort: "medium"`. If that fails too, stop and report to the user. From here the integration branch is frozen: nothing is committed to it again.

9. Push the review branch and open its PR with the Skill tool `pr`, marked as closing the spec and every ticket. The body includes the **review guide**: one section per ticket, in merge order, listing its commits as reported by the splitter, with `test(...)` commits marked skippable.

10. **Review feedback**, whenever the user relays comments: a fresh **implementer subagent** works in a worktree on the review branch, adds fix commits on top titled `fix(review): …` (test-only fixes as `test(review): …`), and pushes. The review branch is never rebased or force-pushed, so review comments stay anchored.

11. Once the PR merges, delete the integration branch and clean up all **implementer subagent** worktrees.
