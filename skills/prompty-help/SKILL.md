---
name: prompty-help
description: >
  Quick reference for prompty-nerdy: the three principles, the nine lint
  rules, which rule fires when several apply, the two severities, the
  report shape, the probe families and the commands. Use when the user
  says "prompty help", "how does prompty work", "what are the prompty
  rules", or invokes /prompty-help.
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

WHICH RULE FIRES
  One input, one finding, under the most specific rule that names the cause:
  no-contradiction > examples-match-rules > disjoint-categories >
  decision-terms-defined > output-contract > no-undecided-input

  no-undecided-input is the catch-all, not the default. The last three rules
  describe different failures and do not compete in that order.

SEVERITY
  error    for some input the prompt does not determine the answer, or it
           determines it by a mechanism that cannot work as written
  warning  determined, but a careful reader could land elsewhere

  always error  no-undecided-input · no-contradiction · examples-match-rules
                disjoint-categories · reason-before-verdict
  splits        decision-terms-defined  error if real data can produce the
                                        splitting input, else warning
                output-contract         error if a parser must guess the shape
                human-executable        error if the reader cannot answer at all
                technique-fits-task     error if the task needs missing work,
                                        warning if an instruction is inert

  A finding needs the input that proves it. No input, no finding — except the
  inert half of technique-fits-task, which takes an experiment, not an input,
  and says so in the finding.

REPORT
  <severity>  <rule>  <where>
              Input: <the input that breaks it>
              <what the prompt determines for it, and why that is a problem>
              Fix: <what closes it>

  Errors first. Input and Fix both required — except the inert half of
  technique-fits-task, the one finding no input can prove, which writes
  "Not proven by an input." in place of the input line.
  Ends with `3 errors, 1 warning`, or the single word `clean`.

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
