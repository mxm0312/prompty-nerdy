---
description: Apply prompt lint fixes and return the full rewritten prompt
argument-hint: "[path or paste the prompt]"
---

Fix this prompt and hand back the complete rewritten version: $ARGUMENTS

If no prompt is given, ask for one in a single line and stop.

First ask, in one line each if the prompt does not say: what consumes the output and what breaks if the shape changes; what decision this makes and what the possible answers are; which inputs it has been getting wrong. The third matters most and nobody volunteers it. If the user does not know, assume every existing instruction is load-bearing and change only what a probe proved broken.

Apply the fixes: give undecided inputs a stated outcome, resolve contradictions by deleting the loser or stating precedence, align demonstrations with the rules (if the demo was right, it carried a rule nobody wrote down — write it down), define terms the answer turns on as checkable conditions, add precedence for overlapping categories, pin the output contract with its allowed values enumerated inline, move any reasoning field ahead of the answer field in the schema itself, and add a technique the task needs or drop an instruction that changes no answer.

There is no required layout. Two constraints are not stylistic: a term is defined before the rule using it, and reasoning precedes the verdict — both because the model reads and generates left to right. Beyond that, shape is a judgment call.

Every fix must be answer-neutral. Before changing a line, name the input whose answer changes; if you can name one, either leave the line alone or report it on its own line as `behavior change: <which inputs now answer differently, and why that is intended>`. Never fold a behavior change into the general findings.

If a term is undefined and cannot be sourced, leave `TODO(define): <term>` rather than inventing one.

Output two blocks: the complete pasteable prompt in a code fence, then one line per fix as `<rule>  <what changed>`, followed by any behavior change lines. No token counts, no percentage saved.
