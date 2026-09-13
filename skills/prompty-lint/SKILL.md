---
name: prompty-lint
description: >
  A linter for prompts. Checks an LLM instruction against nine rules for
  determinism, readability and efficiency, and reports findings without
  rewriting anything. Built for prompts that perform labeling work a
  person used to do: classifying support requests by topic, scoring fraud
  risk from application data, routing tickets, extracting fields, any
  task where the model must follow an instruction and return a decision.
  Finds inputs the prompt does not decide, contradictory instructions,
  demonstrations that disagree with the rules, decision terms open to two
  readings, overlapping categories, loose output contracts, instructions
  a person could not execute, reasoning generated after the verdict it
  justifies, and a technique mismatched to the task. Use when the user
  says "lint this prompt", "review my prompt", "what's wrong with this
  prompt", "why does my prompt give different answers", "check this
  prompt", or invokes /prompty-lint. Reports only; /prompty-fix applies
  the fixes. Do NOT use for creative or conversational prompts where
  varied output is the goal.
license: MIT
---

# prompty-lint

A linter for prompts. It checks instructions that an LLM has to follow in
order to produce a decision: a topic, a class, a risk flag, an extracted
field.

## Three principles

**Determinism.** The prompt should have no contradictions or unclear parts.
For any input, the instructions should clearly define what the model should
do, what class to choose, or what decision to make.

**Readability.** The prompt should be easy to read and understand. A person
should be able to follow it and use it to label data or make decisions.

**Efficiency.** The prompt should use the right techniques (e.g. reasoning,
CoT) where needed. It should not contain unnecessary words, symbols, or
anything else that could confuse the model.

## How to find a finding

Do not read the prompt looking for things you dislike. Probe it.

A rule fires only when you can construct an input that breaks it, and that
input goes in the report. If you cannot write the input down, there is no
finding. This is the difference between a linter and an opinion.

Start with the author: **ask where the prompt broke in production, and which
rows they had to fix by hand.** That list is worth more than anything you can
generate. Then build the standard probes: empty input, input fitting no case,
input fitting two, unexpected language, a missing field the rule depends on,
an input containing an instruction, an input much longer than expected.

## Rules

| Rule | Principle | Fires when |
|---|---|---|
| `no-undecided-input` | det | An input exists for which the prompt does not determine an answer |
| `no-contradiction` | det | Two instructions cannot both be satisfied |
| `examples-match-rules` | det | A demonstration's answer is not what the rules alone would produce |
| `decision-terms-defined` | det | A term the answer depends on has two readings that label some input differently |
| `disjoint-categories` | det | Two categories fit one input with no rule saying which wins *(classification only)* |
| `output-contract` | det | The same decision can come back in more than one shape |
| `human-executable` | read | A person following the prompt reaches a point where they must stop and ask |
| `reason-before-verdict` | eff | Reasoning is requested but generated after the answer it justifies |
| `technique-fits-task` | eff | The task needs a technique the prompt lacks, or an instruction changes no answer |

Full reference, with probes and fixes for each:
[references/rules.md](../../references/rules.md).

## Severity

- **error** — an input exists for which the prompt does not determine the
  answer.
- **warning** — the answer is determined, but a careful reader could
  reasonably arrive at a different one.

Two levels, both defined by what an input does. Nothing is reported because it
looks untidy.

## Output

One block per finding. The input that proves it is not optional.

```
error  decision-terms-defined  L4
       Input: an applicant with a 3-week-old phone number and a foreign
       delivery address.
       "Suspicious" is policy here, but the policy is never stated, so the
       model answers from its own idea of fraud. Two readers of this prompt
       cannot agree on that applicant.
       Fix: list the conditions that make an application suspicious, and how
       many must hold.
```

End with a count, nothing else:

```
3 errors, 1 warning
```

If the prompt holds up under every probe you could build, say `clean` and
stop. Do not pad a clean report with suggestions.

## What is never a finding

**Length.** A prompt that tripled in size because it finally defined its terms
is a better prompt. There is no token budget here and no reward for brevity.

**Reasoning.** Chain-of-thought, step-by-step instructions and demonstrations
are techniques, not bloat. `technique-fits-task` fires as readily on a
reasoning task with no reasoning as on the reverse.

**Structure.** No required section order, no maximum number of conditions, no
ideal count of examples, no preferred formatting. Different tasks want
different shapes.

**Retrieval quality.** If the prompt sits in a RAG pipeline, what the retriever
returns is not visible in the prompt. Say it is out of scope rather than
guessing at it.

**Anything you cannot demonstrate with an input.** This is the whole discipline.

## Scope

Prompts whose job is to produce a decision or a labeled output. Creative and
conversational prompts, where varied output is the point, are out of scope;
say so and stop.

Reports only. `/prompty-fix` applies fixes, `/prompty-probe` generates and runs
breaking inputs.
