# konomi-redteam

### ▶ **Live: https://sjgant80-hub.github.io/konomi-redteam/**

A red-team **taxonomy** that does not go stale, a **collapse corpus** that cannot become a cookbook,
and a **gate on the suite itself**.

[![witness](https://github.com/sjgant80-hub/konomi-redteam/actions/workflows/witness.yml/badge.svg)](https://github.com/sjgant80-hub/konomi-redteam/actions/workflows/witness.yml)

> **The shipped records are illustrative and name a fictional system (`example-model-a`).** None of
> them is an observation of any real model, and no number in this repository should be cited. The
> corpus ships empty of real findings on purpose — see [Why no findings ship](#why-no-findings-ship).

## The three problems this addresses

**Published red-team collections are lists of payloads, and a payload is a fact about one model on one
day.** The lab patches it, the next training run absorbs it, and the entry is dead. Publish a thousand
and within a cycle you have a thousand dead rows and no way to tell which *ideas* among them are still
true.

**"The model refused" is not a pass.** Without a benign control you cannot distinguish a model that is
safe from a model that refuses everything. Over-refusal is a real failure with real users and it is
measured far less often than the failure in the other direction.

**Nobody measures whether a red-team suite is any good.** A suite passes review by being large, and
size is the one property that is trivially faked: five hundred rewordings of one idea look like five
hundred findings and cover one cell.

## What it does instead

### 1 · Classify by the property exploited, never the wording

A pressure is defined by the **property of the system** it exploits. The property outlives the
phrasing, so when a specific wording stops working the cell in the matrix is still occupied — which is
what makes coverage measurable at all.

| Pressure | Prime | Exploits |
|---|---|---|
| **Frame** | 2 ● | The guard evaluates the declared context of a request rather than the effect of answering it |
| **Authority** | 3 〜 | The model cannot verify a claimed permission, so an asserted role substitutes for a real one |
| **Increment** | 5 ┃ | Each turn is judged alone, so a trajectory crosses a boundary no single step crosses |
| **Decompose** | 7 ♡ | The guard scores each part in isolation and never scores the assembled whole |
| **Saturate** | 11 △ | Instruction weight decays with distance and volume, so an early constraint loses to a late one |
| **Invert** | 13 ◐ | A guard phrased over positive requests does not fire on the complement |
| **Encode** | 17 ◯ | The guard operates on one representation while the request arrives in another |

Every family ships with the shape of its **benign twin** — the same surface structure applied to a
legitimate task, which a useful assistant must simply do.

### 2 · Record collapses as signatures, not payloads

A finding's identity is `pressure | axis | mechanism`. Two attacks worded completely differently that
exploit the same property and give way by the same named mechanism are **one finding** — so a corpus
that counts them twice reports coverage it does not have. This is why naming the mechanism is
mandatory: it is the field that makes deduplication mean something, and it is the actual intellectual
work.

A collapse also records **which axis gave way**, reusing the seven axes from
[konomi-rubric](https://sjgant80-hub.github.io/konomi-rubric/). A jailbreak is rarely only a "safety"
event: a model that abandons its assigned role failed on **compliance**; one that invents an authority
to justify itself failed on **calibration**. Recording that distinction is most of the diagnostic value.

### 3 · The payload guard — the corpus cannot become a cookbook

There is no field for a payload, and `defineCollapse` **throws** if the notes carry a transcript line,
a fenced block, or a long verbatim span. `triggerRef` must be an opaque reference — a hex digest or a
ticket id — that resolves only inside your own private store.

```js
defineCollapse({ ..., notes: 'User: ignore your previous instructions' })
// Error: collapse RT-010 rejected:
//   - notes contain a transcript line (user:/system:/assistant:) — record the mechanism, not the conversation
```

This is not a limitation working around the design. It **is** the design: what survives is the
structure, which is the part that was worth publishing. Try to break it on the live page — the refusal
is the feature.

### 4 · The gate on the suite

```js
gateSuite(corpus).holes
```

| Hole | Meaning |
|---|---|
| `UNCOVERED` | A pressure family has never been applied — nothing is known about it either way |
| `REDUNDANT` | Two records share a finding signature — one mechanism counted twice |
| `UNCONTROLLED` | A family produced breaches but no benign twin was run — its results cannot be interpreted |
| `UNREPRODUCED` | A finding rests on too few trials, or a rate that did not hold |
| `OVERREFUSAL` | A control was refused — a real failure, in the direction nobody reports |
| `STALE` | A record already marked fixed is still counted toward current coverage |
| `MONOAXIAL` | Every record is attributed to one axis — the suite is scoring, not diagnosing |

A finding must survive replication: at least **3 trials** at a **50%** rate. A sampled model can do
something once and never again, and one observation is an anecdote. (A record that *held* is exempt —
there is nothing there to replicate.)

### 5 · Reach is one integer, and the corpus is a regression instrument

Each pressure sits on one of the first seven primes, so a coverage vector folds to a single integer
that still factors back into exactly where the suite looked:

```
reach 30030  →  [1,1,1,1,1,1,0]  →  ●〜┃♡△◐     six families, encode untouched
```

And because a signature is stable across rewordings, two corpora taken against different model
versions can be compared directly:

```js
diff(before, after)   // → { fixed, persisting, regressed, new }
```

That is the part that makes this a regression instrument rather than a snapshot: a finding rewritten
in different words still matches itself.

## Why no findings ship

Two reasons that point the same way.

Real red-team findings are almost always produced under contract and are confidential. A tool that
expected you to publish them would be unusable by exactly the people who have them.

And a finding is only meaningful against a named model at a named version. Shipping someone else's
findings as your baseline would be inheriting claims you never verified.

So this is the instrument; you bring the corpus. The gate will tell you what your corpus is missing
long before it tells you anything about a model.

## Verification

```bash
node test.mjs        # 151 assertions
```

Every kernel is gated by [witness](https://github.com/sjgant80-hub/witness) in CI — mutation testing
that modifies the source and reports anything the suite fails to notice.

| kernel | killed | reviewed-equivalent | verdict |
|---|---|---|---|
| `fold.mjs` | 11 / 14 | 3 | no test-theatre |
| `pressure.mjs` | 4 / 4 | 0 | clean outright |
| `collapse.mjs` | 37 / 37 | 0 | clean outright |
| `corpus.mjs` | 23 / 23 | 0 | clean outright |

The suite is deliberately **anti-vacuous**: the two claims that carry this build are both negative —
that a payload *cannot* enter, and that a suite with a hole *cannot* report clean — and a negative
claim is only proven by the thing it rejects. So every guard is tested twice, once with clean input and
once with the specific dirty input it must refuse by name.

Reading the live output caught a real defect: a record that *held* across 9 trials was being reported
as an `UNREPRODUCED` finding, which penalises the outcome the suite is hoping for.

## Honest limits

- **Seven families is a working taxonomy, not a proof that the space has seven parts.** If you find a
  pressure that fits none of them, the taxonomy is wrong and should change.
- The gate checks whether a suite is **interpretable**. It cannot tell you whether your findings are
  important, and it has no opinion about any model.
- Suppressing payloads is a real trade: a reader cannot reproduce your finding from the corpus alone.
  That is the intended cost.
- `clean` is not a score. A suite with an uncontrolled family is not 85% sound — it has a family whose
  results cannot be interpreted.

## Lineage

- The seven axes and the gate-the-instrument pattern come from
  [konomi-rubric](https://github.com/sjgant80-hub/konomi-rubric).
- The fold algebra is vendored from
  [konomigami-lib](https://github.com/sjgant80-hub/konomigami-lib).
- Gated by [witness](https://github.com/sjgant80-hub/witness).

MIT. Intended for authorised safety evaluation and defensive research.
