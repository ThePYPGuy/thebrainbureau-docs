# How an answer leaks without being served

Three ways found on 2026-09-02, none of which `test:answer-leak` can see. It
looks for answer STRINGS in served state, and **none of these serves the
answer** — each hands the child something from which the answer follows.

---

## 1. THE ORACLE — live on a published activity, awaiting a decision

`operation-zero-hour` / `the-value-vault` / `enter-restore-code`. **Published.**

Three links, each verified in the repo:

1. the task is `keypad: true`, answer `{ restoreCode: 7285 }`, tolerance `exact`
2. `lib/engine/validate.ts:28-36` — the `exact` branch, ON A MISS, returns
   `offBy: { kind: "percent", value: pctOff(given, expected) }`
3. `app/api/check/route.ts:38` — `return NextResponse.json(outcome)`, whole

    submit 1000  ->  86.3% off  ->  target approx 7299
    submit 7299  ->   0.2% off  ->  target approx 7284.4

**Three or four submissions against a 10,000-way code**, and *no lockouts* is a
series behaviour, so nothing slows it. `7285` is assembled from four digits
earned across four earlier tasks: **the oracle skips the whole Operation.**

**THE NARROWING IS THE IMPORTANT PART.** Twelve published tasks use `exact` with
a numeric answer and all twelve return `offBy`. **Eleven of them should.**
`validate.ts`'s header says a child learns *roughly how far off — never the
target*, and that this *distinguishes an arithmetic slip from a misunderstood
method*. For *round 5,472 to three significant figures* that is good teaching.

**The header is true literally and false in effect: percent-off plus your own
guess IS the target.** It costs nothing where the child could derive the answer
anyway. It costs everything on a code, **where there is no method to teach.**

*A tolerance says near is worth something. On a code, near is worth everything.*

**DECIDED 2 Sep. Maciej: *"don't show off by when entering a code — that isn't
what the off by mechanic was for — off by is to help students when they are
calculating large numbers."*** Suppress it on code entry, keep it everywhere
else. Prime Directive's seven are calculations and stay as they are.

**He supplied the thing neither the code nor the analysis had: the mechanic's
PURPOSE.** `validate.ts`'s header says what `offBy` DOES — *roughly how far off,
never the target*. He said what it is FOR. **The gap between those two is the
entire bug**: a behaviour correct on a calculation and wrong on a code, with
nothing in the code able to tell which it was looking at.

Two open questions for whoever writes it: whether `config.keypad` is the ONLY
marker for code entry — he said *entering a code*, not *keypad true* — and
whether the condition should key on the code or on the `exact` tolerance, since
a code with a relative tolerance would leak the same way.

---

## 2. ORDER, which a key-name guard cannot see

`assertNoAnswer` reads key NAMES. **An array written in its final order has
shipped the answer** with no forbidden key present. Three instances in one
activity, all passing every test, all found by a person looking at a screen:

- two `matching` tasks whose answer key was the identity mapping of the two
  authored columns — every row beside its own partner
- the finale's `console.route`, authored in `answer.order` exactly, so **1 to 6
  straight down the list won the Operation without reading any evidence**

Every type carrying an ordered list has this hole. The fix is an import-time
assertion handed the config and the answer SEPARATELY, never deriving one from
the other.

---

## 3. THE HINT, which is paid for

A hint costs Intel, so a hint containing its answer charges a child for the
thing they were owed a method for. `findAnswerInHint` exists on `lock-library`
and `scripts/test-hint-leak.ts` runs it **against the DATABASE** — which is where
a child reads it, and where a file-reading check would have been blind.

**It has zero true positives across all four activities**, and two of its three
hits were legitimate: where the EVIDENCE states a value and the task is to apply
it, the hint may repeat it. That is the normal case for map tasks. **A guard
that refuses those refuses good content**, so it must compare against the prompt
before refusing.

---

## The common shape

**Content can leak an answer without containing it.** Adjacency, order,
position, and the shape of a refusal are all answers. **Look at what reaches the
child, not at what is in the keys.**
