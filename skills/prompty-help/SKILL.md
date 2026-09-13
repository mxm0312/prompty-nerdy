---
name: prompty-help
description: >
  Quick reference for prompty-nerdy: the three principles, the nine lint
  rules, the two severities, the probe families and the commands. Use
  when the user says "prompty help", "how does prompty work", "what are
  the rules", or invokes /prompty-help.
license: MIT
---

Print this, nothing else.

```
prompty-nerdy · a linter for prompts

PRINCIPLES
  determinism   for any input, the instruction determines the answer
  readability   a person can follow it and label data with it
  efficiency    right techniques where needed, nothing that confuses the model

RULES                                                          fires when
  det  no-undecided-input      some input has no determined answer
  det  no-contradiction        two instructions cannot both hold
  det  examples-match-rules    a demo disagrees with the rules
  det  decision-terms-defined  a decisive term has two readings
  det  disjoint-categories     two classes fit one input, no precedence
  det  output-contract         one decision, more than one shape
  read human-executable        a person must stop and ask
  eff  reason-before-verdict   reasoning generated after the answer
  eff  technique-fits-task     needed technique missing, or inert instruction

  disjoint-categories applies to classification prompts only.

SEVERITY
  error    an input exists for which the prompt does not determine the answer
  warning  determined, but a careful reader could land elsewhere

  A finding needs the input that proves it. No input, no finding.

PROBE FAMILIES
  empty · fits nothing · fits several · boundary pairs · unexpected form
  missing dependency · embedded instruction · adversarial but plausible

  First question is always: where did it break in production?

NEVER A FINDING
  length · reasoning or demonstrations as such · section order
  retrieval quality · anything you cannot show with an input

COMMANDS
  /prompty-lint    report findings, change nothing
  /prompty-fix     apply fixes, return the full prompt
  /prompty-probe   generate breaking inputs and run them
  /prompty-help    this

  Every fix is answer-neutral, or it is named: "behavior change: ..."
```
