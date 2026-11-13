---
name: from-gaps-to-spec
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

Then list the **gaps**: everything the spec needs that no source settles. Walk the spec's sections (Problem Statement, Solution, Behaviour, Implementation Decisions, Testing Decisions, Out of Scope) and ask what each one would have to say. Every Scope and Done-when bullet of the issue must map to ledger entries or to a gap. Include the test seams here: prefer existing seams to new ones, the highest seam possible, ideally one.

Show the ledger, conflicts and gaps in chat.

## 3. Grill the gaps

Call the Skill tool twice, for "grilling" and "domain-modeling". The conflicts and gaps are the root of the design tree. The ledger is settled ground: its entries enter the grill as given, and you reopen one only when the user does.

Append each answer to the ledger as it settles (`D14 … (grilling)`), so the ledger stays the one record the spec is written from.

Done when the grilling frontier is empty, no conflict or gap remains, and the user confirms the shared understanding.

## 4. Write and publish

Call the Skill tool for "writing-a-spec", with the ledger and the source issue. Once the spec is published, comment on the source issue with a link to it, and report the spec's link.
