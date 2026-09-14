<p align="center">
  <img src="assets/logo.png" width="512" alt="prompty-nerdy">
</p>

<h1 align="center">prompty-nerdy</h1>

<p align="center">
  <em>Feels like a linter for prompts</em>
</p>

---

This skill is useful for prompt engineering tasks where you need to create an
instruction for an LLM that it can follow to perform various tasks.

This could be classifying the topic of a user's support request, assessing
fraud risk based on a person's data, or performing any other type of labeling
that can be replaced with an LLM.

## Three principles

**Determinism** — The prompt should have no contradictions or unclear parts.
For any input, the instructions should clearly define what the model should do,
what class to choose, or what decision to make.

**Readability** — The prompt should be easy to read and understand. A person
should be able to follow it and use it to label data or make decisions.

**Efficiency** — The prompt should use the right techniques (e.g. reasoning,
CoT) where needed. It should not contain unnecessary words, symbols, or
anything else that could confuse the model.

## How it finds problems

A linter that only reads the text finds typos. This one probes.

Every rule is checked by trying to construct an input that breaks it. A finding
is only reported when that input exists and can be written down, so every
report carries the case that proves it:

```
<severity>  <rule>  <where>
            Input: <the input that breaks it>
            <what the prompt determines for that input, and why that is a problem>
            Fix: <what closes it>
```

Exactly one direction of one rule cannot meet that bar. Asking whether a line
changes any answer takes the prompt run with and without it, not a constructed
input, so the inert half of `technique-fits-task` reports as a warning that
says so in the finding itself. Everything else obeys the sentence above.

Probes worth running on any labeling prompt: empty input, input that fits no
category, input that fits two, boundary pairs, input in an unexpected language,
input containing an instruction, input where a field the rule depends on is
missing. Before generating any of them, ask the author where the prompt broke
in production. That list is better than anything you can generate.

## Rules

| Rule | Principle | Fails when |
|---|---|---|
| `no-undecided-input` | determinism | An input exists for which the prompt does not determine an answer |
| `no-contradiction` | determinism | Two instructions cannot both be satisfied |
| `examples-match-rules` | determinism | A demonstration's answer is not what the rules alone would produce |
| `decision-terms-defined` | determinism | A term the decision turns on can be read two ways that yield different answers |
| `disjoint-categories` | determinism | Two categories fit one input and no rule says which wins *(classification only)* |
| `output-contract` | determinism | The same decision can come back in more than one shape |
| `human-executable` | readability | A person following the prompt cannot produce the answer it asks for |
| `reason-before-verdict` | efficiency | Reasoning is requested but generated after the answer it justifies |
| `technique-fits-task` | efficiency | The task needs a technique the prompt doesn't use, or carries instructions that don't affect the answer |

Two severities, defined so they can't drift:

- **error** — for some input the prompt does not determine the answer, or it
  determines it by a mechanism that cannot work as written.
- **warning** — the answer is determined, but a careful reader could reasonably
  arrive at a different one.

The second clause of `error` exists for one rule. Reasoning generated after the
verdict is paid for in full and cannot inform anything, so it is an error even
though the answer is determined. Which side of the line each rule falls on is
in [references/rules.md](references/rules.md#severity).

A run ends with `3 errors, 1 warning`, or `clean`.

When several rules describe the same broken input, the most specific one is the
one reported: `no-contradiction` → `examples-match-rules` →
`disjoint-categories` → `decision-terms-defined` → `output-contract` →
`no-undecided-input`. One input, one finding, because the fix is different in
each case.

## A run

A support-triage prompt, nineteen lines. Four of them:

```
6   - refund — the customer wants their money back
...
15  Example:
16  Message: "I was charged twice for order 4412, please return the extra payment"
17  Answer: billing
```

Two of the eight findings `/prompty-lint` returns:

```
error  examples-match-rules  L16-17
       Input: the demonstration's own message — "I was charged twice for
       order 4412, please return the extra payment".
       The rules put this in refund: the customer wants their money back,
       L6. The demonstration answers billing. The prompt now carries two
       specifications and the demonstration is the one the model follows, so
       every refund request that mentions a charge is labeled billing.
       Fix: decide which is right. If billing is right, L6 is missing the
       rule that a disputed charge stays billing; write it down.

warning  technique-fits-task  L1
         Not proven by an input. Removing the role description changes no
         answer I can construct, but that is an argument, not a result.
         Fix: run both versions over 30 real rows; delete if they agree.
```

The second one is the only finding in the run with no input in it, and it says
so in the report. Showing that a line changes nothing takes the prompt run with
and without it over a set of rows — an experiment, not a probe. `/prompty-fix`
leaves that line alone for the same reason.

`/prompty-fix` returns the rewritten prompt and one line the rest of the report
is not allowed to hide:

```
behavior change: a message that asks for money back and mentions a charge —
"I was charged twice, send it back" — now returns refund where it returned
billing. That was the demonstration's answer and the model was following the
demonstration, so this is the fix landing, not a side effect. If billing was
the intended answer for these, the precedence at rule 1 is the line to reverse.
```

The whole run — the prompt, all eight findings, the rewritten prompt and the
fix report — is in [references/example-run.md](references/example-run.md).

## Install

**Claude Code**

```
/plugin marketplace add mxm0312/prompty-nerdy
/plugin install prompty-nerdy@prompty-nerdy
```

**Any other agent** — paste [AGENTS.md](AGENTS.md) into your system prompt,
rules file, or `CLAUDE.md`. It's the whole ruleset, instruction-only, no
tooling required.

## Commands

| Command | What it does |
|---|---|
| `/prompty-lint` | Report findings. Nothing is rewritten |
| `/prompty-fix` | Apply the fixes and hand back the full prompt |
| `/prompty-probe` | Generate breaking inputs for a prompt and show what it does with them |
| `/prompty-help` | Quick reference |

Every fix is answer-neutral, or it is named. If a change makes some input come
back differently, that change is reported on its own line, never folded into
the rest. A prompt that quietly starts labeling differently is the failure this
skill exists to prevent.

## What's inside

```
skills/          the linter, the fixer, the prober, the reference
commands/        the four slash commands
references/      rules.md        every rule: what it means, how to probe it, how to fix it
                 techniques.md   which technique a task actually needs
                 contract.md     output contract patterns
                 example-run.md  one prompt through lint and fix, nothing trimmed
benchmarks/      empty for now; no benchmark has been run
AGENTS.md        instruction-only ruleset for any agent
```

## Numbers

There are none. No benchmark has been run, and nothing in this repo is
measured — see [benchmarks/](benchmarks/).

## License

MIT.
