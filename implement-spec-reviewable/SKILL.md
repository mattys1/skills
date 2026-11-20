---
name: implement-spec-reviewable
description: "Run /implement-spec, then /make-reviewable-split or -stack on its PR. Usage: /implement-spec-reviewable split|stack"
disable-model-invocation: true
---

You have been given a **method**, `split` or `stack`. If it is missing, ask for it.

1. Read `../implement-spec/SKILL.md` (relative to this skill's base directory) and follow it, with two additions:
   - the user asks for a draft PR, so open one as it describes;
   - the merger subagent merges each ticket with a merge commit (`--no-ff`) titled `Merge <ticket branch> (#<ticket>)`, and the code-review fixes the same way, titled `Merge <branch> (review fixes)`. These merges become the groups in step 2.

   Skip marking the draft PR ready for review: it becomes the source PR for step 2.

2. Read `../make-reviewable-<method>/SKILL.md` and follow it, with the draft PR as the source PR.
