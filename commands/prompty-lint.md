---
description: Lint a prompt for determinism, readability and efficiency
argument-hint: "[path or paste the prompt]"
---

Lint this prompt: $ARGUMENTS

If no prompt is given, ask for one in a single line and stop.

This is a linter for prompts that produce decisions: classification, risk scoring, routing, extraction. Three principles. Determinism: for any input, the instruction determines what the model should do or which class to choose. Readability: a person should be able to follow it and label data with it. Efficiency: the right techniques where needed, and nothing that only confuses the model.

A rule fires ONLY when you can construct an input that breaks it, and that input goes in the report. If you cannot write the input down, there is no finding. Before generating probes, ask the author where the prompt broke in production and which rows they fixed by hand — that list beats anything you generate.

Rules:
- `no-undecided-input` — an input exists for which the prompt determines no answer
- `no-contradiction` — two instructions cannot both be satisfied
- `examples-match-rules` — a demonstration's answer is not what the rules alone would produce
- `decision-terms-defined` — a term the answer depends on has two readings that label some input differently
- `disjoint-categories` — two categories fit one input with no precedence rule (classification prompts only)
- `output-contract` — the same decision can come back in more than one shape
- `human-executable` — a person following the prompt must stop and ask a question
- `reason-before-verdict` — reasoning is requested but generated after the answer it justifies, including a JSON schema that declares the answer key before the reasoning key
- `technique-fits-task` — the task needs a technique the prompt lacks, or an instruction changes no answer

Severity: `error` when an input exists for which the prompt does not determine the answer; `warning` when the answer is determined but a careful reader could reasonably land elsewhere.

Report one block per finding, each with the input that proves it and the fix. End with a count: `3 errors, 1 warning`. If nothing breaks under any probe you could build, say `clean` and stop without padding.

Never a finding: prompt length, the presence of reasoning or demonstrations, section order or structure, retrieval quality, or anything you cannot demonstrate with an input. Do not report token savings — length is not a quality signal here.

Report only. Do not rewrite the prompt.
