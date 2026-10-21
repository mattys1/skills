---
name: decisions-to-spec
description: "Turn an issue that points at decisions (a slice, a map ticket) into a spec: gather every referenced decision, grill the gaps, publish."
disable-model-invocation: true
metadata:
  credits:
    skill: to-spec
    author: Matt Pocock
    url: "https://github.com/mattpocock/skills/blob/main/skills/engineering/to-spec/"
---

The input is an issue that holds few decisions itself and instead points at the places where they were made: linked decision tickets, a parent map, ADRs. You gather everything those references settled into a **ledger**, grill the user on what the ledger leaves open, and publish a spec built from the result.

The issue tracker and triage label vocabulary should have been provided to you. If not, tell the user to run `/setup-matt-pocock-skills`.

## 1. Gather

Read the issue, its comments, and every reference it makes, following each one transitively for as long as it keeps leading to decisions:

- linked issues: the body, plus the resolution comment that closed it;
- the parent map or roadmap: its Decisions so far, and any "changes agreed" section;
- ADRs, `GLOSSARY.md` (or `GLOSSARY-MAP.md`), and `docs/agents/domain.md` if present;
- the code the issue touches: the before-state the spec changes.

Fetch each issue with the tracker's CLI, comments included: resolutions usually live in comments.

Done when every reference has been read and the ones it led to have too.

## 2. Build the ledger

Distill every decision that bears on this issue into a numbered ledger, one decision per entry, closely paraphrased, with its source:

```
D1  Oracle is edge-based Dijkstra over (EdgeID, dir), in test code. (#29 Scope)
D2  Oracle shares only Metric weight reads and the rule store with CRP. (#14)
D3  Path costs compare with exact ==; no path-identity check, because of ties. (#23)
```

When two sources disagree, the later one wins if it explicitly reverses the earlier; record the winner and note the superseded source. A disagreement with no explicit reversal is a **conflict**.

Then list the **gaps**: everything the spec needs that no source settles. Walk the template in step 4 section by section and ask what each one would have to say. Every Scope and Done-when bullet of the issue must map to ledger entries or to a gap. Include the test seams here: prefer existing seams to new ones, the highest seam possible, ideally one.

Show the ledger, conflicts and gaps in chat.

## 3. Grill the gaps

Call the Skill tool twice, for "grilling" and "domain-modeling". The conflicts and gaps are the root of the design tree. The ledger is settled ground: its entries enter the grill as given, and you reopen one only when the user does.

Append each answer to the ledger as it settles (`D14 … (grilling)`), so the ledger stays the one record the spec is written from.

Done when the grilling frontier is empty, no conflict or gap remains, and the user confirms the shared understanding.

## 4. Write and publish

Write the spec from the ledger with the template below. Every ledger entry lands in exactly one section. Cite the source issue in parentheses where a decision came from one, as the ledger does.

Publish it to the issue tracker with the `ready-for-agent` triage label, and comment on the source issue with a link to the spec. Report the spec's link.

<spec-template>

Spec for <source issue>. Decisions come from <every source the ledger cites>.

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
- Which modules will be tested
- Prior art for the tests (i.e. similar types of tests in the codebase)

## Out of Scope

A description of the things that are out of scope for this spec.

## Further Notes

Any further notes about the feature.
</spec-template>
