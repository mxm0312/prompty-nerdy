---
name: prompty-fix
description: >
  Fixes the findings a prompt lint produced and hands back the complete
  rewritten prompt, ready to paste. Applies to instructions that perform
  labeling work: classification, fraud scoring, ticket routing, field
  extraction. Closes undecided inputs, resolves contradictions, aligns
  demonstrations with rules, defines the terms a decision turns on, adds
  precedence for overlapping categories, pins the output contract, moves
  reasoning ahead of the verdict, and adds a technique the task needs.
  Every fix is answer-neutral unless named as a deliberate behavior
  change. Use when the user says "fix this prompt", "rewrite this
  prompt", "apply the fixes", "clean up this prompt", or invokes
  /prompty-fix.
license: MIT
---

# prompty-fix

Apply the fixes. Hand back a prompt, not advice about a prompt.

## Before changing anything

Three questions, answered from the prompt or asked in one line:

1. What consumes the output, and what breaks if the shape changes?
2. What decision is this making, and what are the possible answers?
3. Which inputs has it been getting wrong?

The third is the one that matters most and the one nobody volunteers. It tells
you which rules are load-bearing, and it usually produces a better set of
findings than the lint did.

If the user does not know, assume every existing instruction is there for a
reason and change only what a probe proved broken.

## The rule above the rest

**Every fix is answer-neutral, or it is named.**

Before changing a line, name the input whose answer changes. If you can name
one, either leave the line alone, or report the change on its own:

```
behavior change: <which inputs now answer differently, and why that is intended>
```

Never fold a behavior change into the general findings. A prompt that quietly
starts answering differently is a slow accuracy regression that nobody traces
back to a prompt edit.

## Applying fixes

Work from the findings. Each fix closes the input that produced the finding,
and you should be able to state which one.

- `no-undecided-input` → give the case a stated outcome. Usually a named value
  the prompt returns. `unclear` (not enough information) and `other`
  (decidable, fits nothing listed) are different situations that often route
  differently downstream.
- `no-contradiction` → delete the instruction that loses, or state which wins.
  Never soften both into something vague.
- `examples-match-rules` → decide which side is wrong. If the example was
  right, it was carrying a rule nobody wrote down; write it down.
- `decision-terms-defined` → state the term operationally, as conditions a
  person could check. If the definition is genuinely unknown, leave
  `TODO(define): <term>` rather than inventing one.
- `disjoint-categories` → separate the definitions, or state precedence.
  Precedence is often the honest answer.
- `output-contract` → pin the shape, the allowed values, how many, and that
  nothing else is emitted. Enumerate values inline in the contract.
- `human-executable` → remove whatever made the reader stop and ask.
- `reason-before-verdict` → move the reasoning field ahead of the answer field
  in the schema itself, not only in the prose.
- `technique-fits-task` → add the technique the task needs; remove the
  instructions that change no answer.

## Structure

There is no required layout. Arrange the prompt however the task reads best,
with two constraints that are not stylistic:

- A term is defined before the rule that uses it, because the model reads once,
  left to right.
- Any reasoning field precedes the answer field, for the same reason.

Beyond those, shape is a judgment call. Do not impose numbered steps on a task
that does not branch, or a decision table on three sentences of policy.

## Output

Two blocks:

1. The complete prompt in a code fence. Pasteable, no `[rest unchanged]`.
2. One line per fix: `<rule>  <what changed>`. Then any `behavior change:`
   lines, each on its own.

No token counts, no percentage saved. Length is not the result.

## Scope

Prompts whose job is to produce a decision or a labeled output. Creative and
conversational prompts are out of scope; say so and stop.
