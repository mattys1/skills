---
name: writing-a-spec
description: "Write a spec from a decision ledger and publish it. Called by the spec skills."
user-invocable: false
metadata:
  credits:
    skill: to-spec
    author: Matt Pocock
    url: "https://github.com/mattpocock/skills/blob/main/skills/engineering/to-spec/"
---

The calling skill has built a **ledger**: numbered, settled decisions (`D1`, `D2`, …), each with its source. You turn it into a spec and publish it. The ledger is the whole record: the spec says what the ledger says, in the template's shape.

The issue tracker and triage label vocabulary should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`.

## 1. Write

Fill the template below from the ledger. Every ledger entry lands in exactly one section. Cite the source in parentheses where a decision came from an issue or ADR, as the ledger does.

Done when every entry is placed and Behaviour covers every rule the ledger implies, edge cases included.

## 2. Publish

Publish the spec to the issue tracker with the `ready-for-agent` triage label. Report the spec's link to the calling skill's next step, or to the user if there is none.

<spec-template>

Spec for <source issue, or the topic of the conversation>. Decisions come from <every source the ledger cites>.

## Problem Statement

What is wrong or missing today, from the user's perspective, measured against the decisions.

## Solution

The solution, from the user's perspective, in a few paragraphs.

## Behaviour

A numbered list of observable rules: what a caller, user or test sees through the seam. One rule per line, stated as a fact, with its reason only where the reader couldn't infer it:

1. A `Directed` Edge is ∞ backward for car.
2. An unknown access value falls back to the next less specific key, so a typo can't make a road impassable.

Exhaustive: every behaviour the ledger implies, edge cases included.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts, not a working demo, just the important bits.

## Testing Decisions

A list of testing decisions that were made. Include:

- A description of what makes a good test (only test external behavior, not implementation details)
- Which modules will be tested, through which seams
- Prior art for the tests (i.e. similar types of tests in the codebase)

## Out of Scope

A description of the things that are out of scope for this spec.

## Further Notes

Any further notes about the feature.
</spec-template>
