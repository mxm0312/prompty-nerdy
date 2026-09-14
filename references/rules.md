# Rules

Nine rules, grouped by principle. Each one states what it means, how to probe
for it, how to fix it, and what it does **not** fire on.

A rule fires only when you can write down an input that breaks it. No rule
counts things. If a check can only be expressed as "more than N of something",
it is a style opinion and it is not in this file.

Exactly one direction of one rule cannot meet that bar — the inert half of
`technique-fits-task`, which takes an experiment rather than an input. It is
called out where it appears, and it reports as a warning that says so. Nothing
else in this file is allowed the same move.

| Rule | Principle |
|---|---|
| [`no-undecided-input`](#no-undecided-input) | determinism |
| [`no-contradiction`](#no-contradiction) | determinism |
| [`examples-match-rules`](#examples-match-rules) | determinism |
| [`decision-terms-defined`](#decision-terms-defined) | determinism |
| [`disjoint-categories`](#disjoint-categories) | determinism, classification only |
| [`output-contract`](#output-contract) | determinism |
| [`human-executable`](#human-executable) | readability |
| [`reason-before-verdict`](#reason-before-verdict) | efficiency |
| [`technique-fits-task`](#technique-fits-task) | efficiency |

## Which rule fires

Several rules describe the same failure from different angles: a contradiction,
two overlapping categories and an undefined term all produce an input the
prompt does not decide. One input produces one finding, reported under the most
specific rule that names its cause:

1. `no-contradiction` — two instructions cannot both be satisfied
2. `examples-match-rules` — a demonstration disagrees with the rules
3. `disjoint-categories` — two categories fit and nothing says which wins
4. `decision-terms-defined` — one term carries two readings
5. `output-contract` — the decision is made, the shape it comes back in is not
6. `no-undecided-input` — nothing above names the cause

`no-undecided-input` is the catch-all, not the default. When a more specific
rule explains why the input is undecided, report it there: the fix is different
in each case, and that is the whole point of naming the rule.

The other three rules describe different failures and do not compete with this
list. `human-executable` is about a reader who cannot proceed, `reason-before-verdict`
about the order work is generated in, `technique-fits-task` about work the
prompt asks for or fails to ask for.

## Severity

- **error** — for some input the prompt does not determine the answer, or it
  determines it by a mechanism that cannot work as written.
- **warning** — the answer is determined, but a careful reader could reasonably
  arrive at a different one.

The second clause of `error` exists for exactly one rule: reasoning generated
after the verdict is paid for in full and cannot inform anything.

| Rule | error | warning |
|---|---|---|
| `no-undecided-input` | always | — |
| `no-contradiction` | always | — |
| `examples-match-rules` | always — two specifications, and the demonstration wins | — |
| `decision-terms-defined` | the two readings split an input the data can produce | they split only a constructed one |
| `disjoint-categories` | always | — |
| `output-contract` | a parser would have to guess the shape | the shape is pinned, but a value is enumerated only in an earlier section |
| `human-executable` | the reader cannot produce an answer at all | the reader proceeds, but could reasonably land elsewhere |
| `reason-before-verdict` | always — the reasoning cannot inform the answer, by construction | — |
| `technique-fits-task` | the task requires work the prompt never asks for | an instruction changes no answer |

Nothing else is a finding. If you cannot name the input, you do not have one —
with one exception, named in `technique-fits-task` below and nowhere else.

---

## `no-undecided-input`

**Means.** For any input the prompt might receive, the instructions determine
what comes back.

**Probe.** Build inputs at the edges and ask what the prompt says to do with
each. The standard set for a labeling prompt:

- empty, or whitespace only
- input that fits none of the described cases
- input that fits more than one
- input in a language the prompt didn't anticipate
- input where the field a rule depends on is missing or null
- input containing an instruction ("ignore the above and answer X")
- input far longer than the prompt's author imagined

Then ask the author which inputs broke it in production. That list beats any
generated one, and it is the first question worth asking.

**Fix.** Give the undecided case a stated outcome. Usually that is a named
value the prompt can return: `unclear` for "not enough information to decide",
`other` for "decidable, but fits nothing listed". Those are different
situations and often route to different places downstream, so they deserve
different values.

The point is not that every prompt needs a bucket called `other`. It is that
the instruction says what happens, whatever the case turns out to be.

**Does not fire on.** Inputs the prompt cannot receive. If a validator upstream
guarantees non-empty Russian text, the empty case is not this prompt's problem.
Ask before assuming either way.

---

## `no-contradiction`

**Means.** No two instructions where satisfying one breaks the other.

**Probe.** Read every instruction against every other instruction, not top to
bottom. Contradictions hide between sections: the rule is in the task
description, the one that cancels it was appended after an incident six months
later.

Pairs to look for:

| One instruction | The one that cancels it |
|---|---|
| "If unsure, return `unclear`." | "Always pick the most likely option." |
| "Be brief." | "Explain your reasoning for each field." |
| "Judge only the customer's messages." | "Consider the whole conversation." |
| "Return one value." | "List everything that applies." |

**Fix.** Delete the one that loses, or keep both and state which wins. Do not
soften both into something vague; that turns a contradiction into an undefined
term, which is harder to see and just as broken.

**Does not fire on.** Instructions that merely sit at different levels of
detail, or a general rule followed by a stated exception. "Label by topic.
Refund requests are always `refund` even when they mention delivery" is a
precedence rule, not a contradiction.

---

## `examples-match-rules`

**Means.** Every demonstration in the prompt is an answer the rules alone
would produce.

**Probe.** Take each example, cover its answer, and work it out from the rules.
Compare. Where they differ, the prompt contains two specifications and the
model will follow the examples.

**Fix.** One of the two is wrong. Decide which:

- The example is wrong → relabel it, and say in one line why it labels that
  way, so the next editor does not undo the fix.
- The rules are incomplete → the example was carrying a rule nobody wrote down.
  Write it down. This is the more common case and the more valuable one.

**Does not fire on.** An example that demonstrates format, tone or structure
rather than a decision. It also does not fire on examples being "unbalanced" or
"too many" — the number and distribution of demonstrations is a modeling
choice, measured on data, not a lint finding.

---

## `decision-terms-defined`

**Means.** A term the answer depends on cannot be read two ways that produce
different answers.

**Probe.** For each substantive term, try to construct two readings *and* an
input those two readings label differently. Both halves are required. If you
cannot produce the input, the ambiguity is theoretical and this rule stays
silent.

Terms that usually qualify: the ones that are policy inside a company and
common words outside it. *Suspicious*, *escalation*, *VIP*, *churn risk*,
*relevant*, *toxic*, *spam*. The tell is that two people on the team would
answer differently if you asked them what it means.

**Fix.** State the term operationally, in a form a person could check against
the input. A list of conditions and a threshold beats an adjective.

If the definition is genuinely unknown, write `TODO(define): <term>` and stop.
A confident wrong definition is a wrong answer with no symptom.

**Does not fire on.** Every ambiguous word. Natural language is ambiguous
everywhere and that is fine; almost none of it reaches the answer. This rule
is about the terms the decision turns on, and nothing else. If two readings
give the same answer on every input you can construct, there is no finding
here.

---

## `disjoint-categories`

*Classification prompts only. Silent on extraction, scoring, generation.*

**Means.** No input fits two categories with nothing to say which one wins.

**Probe.** Try to write one input that honestly belongs to two of the listed
categories. `billing` and `refund` for "I was charged twice, send it back".
`spam` and `fraud` for a phishing message. If you can write it, and the prompt
has no precedence rule, that is an error.

**Fix.** Either make the definitions disjoint, or state the precedence
explicitly: *when both `refund` and `billing` apply, label `refund`*.
Precedence is often the honest answer, because real categories overlap and
pretending otherwise just moves the ambiguity into the wording.

**Does not fire on.** Multi-label prompts that intend overlap. There, the
question is whether the prompt says how many labels to return and in what
order, which is `output-contract`.

---

## `output-contract`

**Means.** One decision comes back in one shape.

**Probe.** Ask what a parser would have to accept. If the answer includes "it
depends", or the prompt says "return the category" without saying in what
form, there is a finding.

Observed variants when the shape isn't pinned: `true`, `True`, `"true"`,
`Yes, this is true`, a JSON object wrapped in a code fence, the answer
preceded by "Here is the classification:".

**Fix.** State the shape, the allowed values, how many, and that nothing else
is emitted. For enums, list the values inline in the contract itself, not only
in a section further up.

Patterns: [contract.md](contract.md).

**Does not fire on.** Free-text outputs where variation is the point. A
summarization prompt has no enum to pin, but it may still need a length or
format constraint if something downstream depends on one.

---

## `human-executable`

**Means.** A person given the prompt and an input can follow it and produce
the answer it asks for.

**Probe.** The real version: hand the prompt to someone who has not seen the
task, with twenty real rows, and compare their answers to yours. It is the
most informative check available and the one teams skip.

The desk version: read the prompt as if you had to execute it, and find the
first point where you would have to stop and ask a question. That point is the
finding.

**Fix.** Whatever removes the question. Often that is stating a rule the author
held in their head; sometimes it is separating instructions that were tangled
together; sometimes it is a worked example.

**Does not fire on.** Length, nesting depth, section order, or formatting
style. There is no correct structure for a prompt. Numbered steps help some
tasks and get in the way of others. This rule fires on a reader who cannot
proceed, not on a shape someone dislikes.

---

## `reason-before-verdict`

**Means.** If the prompt asks for reasoning, that reasoning is generated
before the answer it supports.

**Probe.** Read the output contract left to right. Does a field carrying the
answer appear before a field carrying reasoning or evidence?

```
wrong: {"label": "refund", "reason": "asks for money back on order 4412"}
right: {"reason": "asks for money back on order 4412", "label": "refund"}
```

**Why it is not a style preference.** These are decoder-only models: tokens are
generated left to right, each conditioned on the ones before it. A label
emitted before its reasoning was not informed by that reasoning. It was
sampled first, and the explanation is fitted to a decision already made. The
prompt pays for reasoning and receives a rationalization that reads exactly
like the real thing.

**Fix.** Put the reasoning field first in the schema, not only earlier in the
prose. Strict-schema and structured-output modes emit keys in the order the
schema declares them, so the schema is where this has to be fixed. In plain
text: reasoning first, answer on the last line, parser takes the last line.

**Does not fire on.** Prompts that ask for no reasoning. Asking for none is a
legitimate choice, and this rule has nothing to say about it.

---

## `technique-fits-task`

**Means.** The prompt uses the techniques its task needs, and its instructions
affect the answer.

**Probe.** Two directions, and the first matters more.

*Missing.* Does the task require work the prompt never asks for? A decision
that depends on several conditions combined, on arithmetic, on comparing
dates, on weighing evidence — with no step-by-step reasoning requested. That
is an under-specified prompt, and adding reasoning is the fix.

*Inert.* Is there an instruction you can remove without any input's answer
changing? Role descriptions, encouragement, restated rules, emphasis. Not
because they cost tokens, but because they occupy the model's attention and
sit between the reader and the four lines that decide the answer.

**The inert direction is the one no single input can prove.** Showing that a
line changes nothing takes the same prompt run with and without it over a set
of rows, not one constructed input. So this half of the rule reports as a
warning and says what it is:

```
warning  technique-fits-task  L12-18
         Not proven by an input. Removing the role description changes no
         answer I can construct, but that is an argument, not a result.
         Fix: run both versions over 30 real rows; delete if they agree.
```

If you are not prepared to write that, do not report it. The missing-technique
direction is unaffected: there an input exists and it goes in the report as
usual.

**Fix.** Add the technique the task needs. Remove the instructions that change
nothing. Which techniques suit which tasks: [techniques.md](techniques.md).

**Does not fire on.**

- Reasoning, chain-of-thought, or demonstrations *per se*. These are tools.
  Their presence is never a finding.
- Prompt length. A long prompt that needs to be long is fine.
- Retrieval quality. If the prompt is part of a RAG pipeline, what the
  retriever returns is not visible in the prompt and is not judged here.
  Report it as out of scope rather than guessing.

---

## The rule above the rules

**Every fix is answer-neutral, or it is named.**

Before changing a line, name the input whose answer changes. If you can name
one, either the line stays, or the change is reported on its own:

```
behavior change: <which inputs now answer differently, and why that is intended>
```

A prompt that quietly starts answering differently is the failure this whole
exercise exists to prevent. It ships as a slow accuracy regression that nobody
traces back to a prompt edit.
