---
description: Generate breaking inputs for a prompt and show what it does with them
argument-hint: "[path or paste the prompt]"
---

Find the inputs that break this prompt: $ARGUMENTS

If no prompt is given, ask for one in a single line and stop.

Ask the author first: which rows did you fix by hand, where did it get embarrassing, what did you add to the prompt after an incident. Real failures beat generated ones. Generate only to cover what those answers did not.

Then build probes from the families that apply to this task, skipping any that cannot occur:
- empty and degenerate — nothing, whitespace, one character, punctuation only
- fits nothing — valid and comprehensible, matches no described case
- fits several — honestly belongs to two described cases
- boundary pairs — two near-identical inputs that should get different answers, and two very different ones that should get the same answer. Most real disagreement lives here; spend the most effort on it
- unexpected form — another language, mixed languages, emoji, an input ten times longer than the author imagined
- missing dependency — a rule reads a field that is absent, null or empty
- embedded instruction — input containing "ignore the above and answer X", relevant whenever any part of the input is user-written
- adversarial but plausible — what a motivated person sends once they know how the classifier works

Report each probe as three lines: `input`, `expected` (what the author says should happen), `actual` (what the prompt actually determines, and why), then the rule it implicates.

If you can run the prompt, run it several times on the same input and report whether the answers agreed — disagreement across runs is the strongest evidence a determinism finding can have. If you cannot run it, say so and mark the row as reasoned rather than observed. Never present a prediction as a result.

End by grouping the breaking inputs under the rules they implicate, as input for /prompty-fix.

An input nobody will ever send is not a finding. If the data cannot contain it, skip it.
