# phi-model

## Idea

A model covered by a HIPAA BAA costs far more than a regular model. Mixing the two could lower the overall cost, but only if PHI never reaches the model without the BAA.

This project builds the **test** for that: a checker that reads the full transcript sent to a model and classifies it as **PHI** or **not PHI**. The goal is to measure whether any routing solution works or fails, not to build the router.

PHI is the default. A transcript goes to the cheaper model only if it passes as not PHI. A false positive only costs money; a false negative is a breach.

## Phase 1: Legal definition → concrete rules

Turn HIPAA's vague definition of PHI into concrete rules that are **at least as strict as the law**, and as close to it as possible. A domain expert then approves them.

- **Start from Safe Harbor** (45 CFR 164.514(b)(2)): the 18 identifier types plus the "no actual knowledge" condition. Fill in the gaps it leaves: rare details, combinations of facts, bare client names, free text, and internal IDs.
- **Each rule has five parts:**
  1. the legal text it comes from
  2. a concrete test with no judgment calls
  3. where it's stricter than the law, and why
  4. examples that pass and examples that fail
  5. whether a script can check it or it needs the AI judge
- **Check whether the rules are concrete enough.** Two independent reviewers apply them to the same transcripts. Any rule they disagree on gets rewritten.
- **Track the gap:** the cases a rule blocks that the law would likely allow.
- **The expert approves labeled examples, not just the wording.** Those examples become the test set for Phase 2.
