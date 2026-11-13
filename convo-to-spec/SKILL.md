---
name: convo-to-spec
description: "Turn the current conversation (a grilling session, a design chat) into a spec and publish it."
disable-model-invocation: true
metadata:
  credits:
    skill: to-spec
    author: Matt Pocock
    url: "https://github.com/mattpocock/skills/blob/main/skills/engineering/to-spec/"
---

The decisions were already made in this conversation. You **synthesize** them into a ledger and publish a spec from it; the interview is over.

## 1. Ground

Explore the code the conversation touches, if you haven't already, plus ADRs in that area, `GLOSSARY.md` and `docs/agents/domain.md` if present. Use the glossary's vocabulary throughout; respect the ADRs.

## 2. Build the ledger

Distill every decision the conversation settled into a numbered ledger, one decision per entry, closely paraphrased, with its source: `(conversation)`, or the issue or ADR it came from.

```
D1  Oracle is edge-based Dijkstra over (EdgeID, dir), in test code. (conversation)
D2  Oracle shares only Metric weight reads and the rule store with CRP. (#14)
```

Where the conversation changed its mind, the later word wins; record only the winner.

Add the test seams: prefer existing seams to new ones, the highest seam possible, ideally one.

Done when every decision in the conversation, from the first message to the last, has an entry.

## 3. Check

Show the ledger and the seams in chat, along with anything the spec would need that the conversation left open. Confirm with the user that the seams match their expectations, and let them settle the open points in one reply. Add what they say to the ledger.

## 4. Write and publish

Call the Skill tool for "writing-a-spec", with the ledger. Report the spec's link.
