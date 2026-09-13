# prompty-nerdy — a linter for prompts

Instruction-only ruleset. Drop this into any agent that does not load skills.

Applies to prompts that perform labeling work a person used to do: classifying
support requests by topic, scoring fraud risk from application data, routing
tickets, extracting fields. Not to creative or conversational prompts, where
varied output is the point.

## Three principles

**Determinism** — The prompt should have no contradictions or unclear parts.
For any input, the instructions should clearly define what the model should do,
what class to choose, or what decision to make.

**Readability** — The prompt should be easy to read and understand. A person
should be able to follow it and use it to label data or make decisions.

**Efficiency** — The prompt should use the right techniques (e.g. reasoning,
CoT) where needed. It should not contain unnecessary words, symbols, or
anything else that could confuse the model.

## How to find a finding

Do not read the prompt hunting for things you dislike. Probe it.

A rule fires only when you can construct an input that breaks it, and that
input goes in the report. No input, no finding.

Ask the author first: where did this break in production, which rows did you
fix by hand? That beats anything you generate. Then cover what is left with
probes — empty input, input fitting no case, input fitting two, boundary pairs
(two near-identical inputs that should answer differently, two very different
ones that should answer the same), unexpected language, a missing field a rule
depends on, input containing an instruction, input far longer than expected.

## Rules

- `no-undecided-input` — an input exists for which the prompt determines no
  answer. Fix by giving the case a stated outcome. `unclear` (not enough
  information) and `other` (decidable, fits nothing listed) are different
  situations and often route differently.
- `no-contradiction` — two instructions cannot both be satisfied. Delete the
  one that loses or state which wins; never soften both into something vague.
- `examples-match-rules` — a demonstration's answer is not what the rules alone
  would produce. When they disagree, the demonstration wins and the rules
  become decoration. If the demonstration was right, it was carrying a rule
  nobody wrote down: write it down.
- `decision-terms-defined` — a term the answer depends on has two readings that
  label some input differently. Requires both a second reading *and* an input
  they split on. Not every ambiguous word; only the ones the decision turns on.
- `disjoint-categories` — two categories fit one input and nothing says which
  wins. Classification prompts only. Fix by separating the definitions or
  stating precedence; precedence is often the honest answer.
- `output-contract` — the same decision can come back in more than one shape.
  Pin the shape, the allowed values enumerated inline, how many, and that
  nothing else is emitted.
- `human-executable` — a person following the prompt reaches a point where they
  must stop and ask. Fix whatever produced the question.
- `reason-before-verdict` — reasoning is requested but generated after the
  answer it justifies. Decoding is left to right, so a label emitted before its
  reasoning was not informed by it; the explanation is fitted to a decision
  already made. Fix in the schema, since strict-schema modes emit keys in
  declared order.
- `technique-fits-task` — the task needs a technique the prompt lacks, or an
  instruction changes no answer. The first direction matters more.

## Severity

- **error** — an input exists for which the prompt does not determine the answer.
- **warning** — the answer is determined, but a careful reader could reasonably
  arrive at a different one.

Report the input that proves each finding. End with a count: `3 errors, 1
warning`, or `clean`.

## Never a finding

Prompt length. The presence of reasoning, chain-of-thought or demonstrations.
Section order, nesting depth, or any count of conditions or examples. Retrieval
quality, which is not visible in the prompt. Anything you cannot demonstrate
with an input.

Do not report token savings. Length is not a quality signal: a prompt that
tripled in size because it finally defined its terms is a better prompt.

## When fixing

Every fix is answer-neutral, or it is named. Before changing a line, name the
input whose answer changes. If you can name one, either leave the line alone or
report it on its own line as `behavior change: <which inputs now answer
differently, and why that is intended>`. Never fold one into the general
findings.

There is no required layout. Two constraints are not stylistic: a term is
defined before the rule that uses it, and reasoning precedes the verdict — both
because the model reads and generates left to right.

If a term is undefined and cannot be sourced, write `TODO(define): <term>`
rather than inventing one. A confident wrong definition is a wrong answer with
no symptom.

(This file applies to agents working on the prompty-nerdy repo too.)
