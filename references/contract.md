# Output contracts

Reference for [`output-contract`](rules.md#output-contract). A prompt without
one can return the same decision in more than one shape, and something
downstream has to guess which it got.

## The five things a contract pins

1. **Shape** — JSON object with named keys and types, or a single bare token.
2. **Values** — the closed set, enumerated inline.
3. **Cardinality** — exactly one, zero or more, at most three.
4. **Fallback** — what comes back when nothing fits. This is where
   [`no-undecided-input`](rules.md#no-undecided-input) and the contract meet:
   `unclear` (not enough information to decide) and `other` (decidable, fits
   nothing listed) are different situations and usually route differently.
5. **Nothing else** — no preamble, no explanation, no code fence, no trailing
   note.

Miss any one and you have a contract that holds until the first unusual input.

## Patterns

### Single label, the default for classification

```
Return exactly one word from this list and nothing else:
billing, refund, delivery, account, other, unclear
```

Cheapest to parse, cheapest to generate, hardest to get wrong. Use it unless
you actually need more.

### Structured, when downstream needs fields

```
Return one line of JSON, nothing before or after it:
{"topic": "<billing|refund|delivery|account|other|unclear>",
 "amount_rub": <integer or null>,
 "confidence": <float 0.00-1.00>}
```

Every key typed. Every enum enumerated. `null` explicitly allowed where the
field is optional, otherwise the model invents a placeholder string.

### Multi-label

```
Return a JSON array of 0 to 3 topics from the list, most relevant first:
["refund", "delivery"]
Empty array if none apply. Never repeat a value.
```

State the ordering rule and the cap. Without a cap, recall creeps up over the
tail of the distribution and your precision quietly rots.

### With a reason, when a human reviews the output

```
Return one line of JSON:
{"label": "<enum>", "evidence": "<verbatim quote from the input, max 15 words>"}
```

`evidence` as a verbatim quote, not free-form reasoning. A quote is
checkable — you can assert it appears in the input. A rationale is prose the
model can confabulate at no cost, and reviewers trust it anyway.

Note the key order. It is not cosmetic — see below.

## Key order: thinking before verdict

The most expensive one-character mistake in this file is putting the answer
key first.

Decoder-only models generate left to right, each token conditioned on the
tokens already emitted. A `label` emitted before a `reason` was not informed
by that reason: the label was sampled first and the explanation is written to
fit it. The output looks identical to real reasoning and is worth nothing,
except that it now also misleads whoever reviews it.

```
wrong: {"label": "refund", "reason": "customer asks for money back on order 4412"}
right: {"reason": "customer asks for money back on order 4412", "label": "refund"}
```

Rules that follow from it:

- Any thinking field — `reason`, `steps`, `analysis`, `evidence` — is declared
  **before** the field carrying the answer, and the contract says the key
  order is exact.
- **Strict-schema and structured-output modes do not fix this.** They emit
  keys in the order the schema declares, so a schema with `label` first
  produces a label generated first. Order the schema, not just the prose.
- In plain-text output, same law: reasoning first, answer on the final line,
  parser takes the last line. "Give the answer, then explain" is a request for
  rationalisation.
- If the reasoning field is not worth generating before the answer, it is not
  worth generating at all. Reasoning after the verdict is the only variant
  that is paid for in full and returns nothing.

Sanity check on any existing prompt: read the schema left to right and ask
whether a human forced to write it in that order could do the job. If they
would have to decide before thinking, so does the model.

## Confidence fields

A self-reported `confidence` float is not a probability. It is a token the
model likes emitting, weakly correlated with correctness, and it clusters on
0.85 and 0.95.

Use it only if you have measured that it separates your errors, and calibrate
the threshold on your own data. Otherwise ask for a coarse enum you can
actually validate:

```
"certainty": "<clear|borderline>"
```

Two buckets that a human annotator would also agree on beat a spurious float
with two decimal places.

## `other` versus `unclear`

Not the same thing, and collapsing them costs you the ability to act on
either:

- `other` — the input is clear, it just does not fit any listed category.
  Signal: your label set is missing a class. Action: review the `other` pile
  monthly and split out what has grown.
- `unclear` — there is not enough information to decide. Signal: bad input.
  Action: route to a human, or drop the row.

Separate them and each one becomes a metric you can watch. Merge them and you
get a bucket that grows for two unrelated reasons and tells you nothing.

## Failure modes to write against

| Failure | Guard |
|---|---|
| Model wraps JSON in ```` ```json ```` | "Return raw JSON with no code fence." |
| Model prefixes "Here is the classification:" | "The first character of your reply is `{`." |
| Model invents a category | Enumerate the closed set inline in the contract, not only in an earlier section. |
| Model returns two labels when you wanted one | State cardinality explicitly: "exactly one". |
| Model returns `"N/A"`, `"none"`, `"unknown"` interchangeably | Give the fallback a single spelled-out value and put it in the enum. |
| Empty optional fields become `""`, `"null"`, `"-"` | "Use JSON `null`, never an empty string." |

Each guard is one line, and each one closes a shape the answer could
otherwise arrive in. They look like clutter and are not.

## Placement

Put the contract where the model will still have it when it answers, which in
practice means near the end rather than in an opening paragraph that the rest
of the prompt buries.

This is a tendency, not a rule, and it is not linted. Prompts differ, and a
contract repeated in two places works fine. What *is* linted is whether the
shape is pinned at all.
