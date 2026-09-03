# How an answer leaks without being served

Three ways found on 2026-09-02, none of which `test:answer-leak` can see. It
looks for answer STRINGS in served state, and **none of these serves the
answer** — each hands the child something from which the answer follows.

---

## 1. THE ORACLE — CLOSED 3 Sep at `227465d`, by two independent gates

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

### HOW IT WAS ACTUALLY CLOSED, and why it took two gates

**Both fixes exist and both are needed.** `main` carried the narrow one keyed on
`config.keypad`; `lock-library` carried the wide one keyed on MAGNITUDE. They
were written independently, against the same defect, and **collided on the same
lines of `withinTolerance` when the branches merged** — which is how the second
one came to light at all.

Neither alone is enough:

    magnitude alone   `restoreCode` is 7285, well above the floor, so the
                      magnitude gate CANNOT withhold it
    keypad alone      four other published tasks carry one digit each and
                      none of them sets `keypad`

**And they were proved independent by falsification, not by reading.** Forcing
`distanceIsSafe` to return `true` and re-probing:

    large code, magnitude gate disabled
      raw offBy            {"kind":"percent","value":86.3}
      keypad gate applied  undefined      <- still withheld
    large calculation      {"kind":"percent","value":100}   <- still helps

So neither gate is the other wearing a second coat. **A second gate derived from
the first gate's source is one gate**, and the only way to know which you have is
to break one and watch the other.

The last line matters as much as the first: the mechanic still works where
Maciej said it was for. *Off by is to help students when they are calculating
large numbers* — and it still does.

**One behaviour changed beyond the leak.** The `exact` arm no longer returns an
absolute distance when the expected value is zero. Measured across all published
activities: **25 exact numeric fields, none answering zero**, so nothing live is
affected. A future task answering zero will get no distance at all.

### THE FIX CLOSED THE SLOW ROUTE AND LEFT THE FAST ONE OPEN

**`fix-code-oracle` is cut, verified and NOT complete.** Do not merge it and
call the oracle closed.

`isCodeEntry` keys on `config.keypad === true`, which appears exactly once in
all content. But **four other published Zero Hour tasks each carry one digit of
the restore code**, none has `keypad`, and all still return `offBy` per field:

    unjam-the-archive        {valueA, positionB, digit1: 7}
    lock-the-telescope       {roundA, signal,    digit2: 2}
    stabilise-the-gauges     {g1,g2,g3,g4,       digit3: 8}
    decode-the-inscriptions  {yearA, yearB, difference, digit4: 5}

Measured with the engine's own functions, **one probe of `1` into every field**
recovers all four digits exactly. **Four probes, four tasks, the whole code, and
the vault is never touched.** `stabilise-the-gauges` surrenders five fields in a
single POST. The vault was the SLOW route.

### THE AXIS IS MAGNITUDE, AND MACIEJ NAMED IT FIRST

*"off by is to help students when they are calculating large numbers."* The
measurement agrees with him exactly:

    roundA 4400000 -> recovered null     signal 3720 -> null
    digit1 7 -> 7      g1 28 -> 28       difference 175 -> 167

**Large numbers do not leak; small ones surrender exactly.** With `pctOff`
rounded to one decimal, a large value's percentage is coarse relative to its
magnitude while a single digit is fully determined. **The mechanic is safe
precisely where he says it is useful and unsafe everywhere else.**

So the condition wants to be **per-FIELD and on magnitude**, with `keypad` kept
as a second independent reason — `restoreCode` is 7285, and magnitude alone
would not withhold it.

**Open, and Maciej's: it is a policy change on published content.**

### And a content-design property that no fix to `offBy` reaches

**Each of the four digits is recoverable from the task that produces it.** So
even with `offBy` entirely closed, a child who can read four tasks' responses
assembles the restore code without doing the fourth task. That is how the
Operation is authored rather than a defect in the engine, and it is worth
knowing before the next Operation reuses the shape.

### How the narrow answer happened, because it is the useful part

I asked whether `keypad` was the only marker for code entry. It was, and that
answer was correct. **The question that mattered was not *which tasks are code
entry* but *which answers does a distance give away*, and those are different
sets.** A well-answered narrow question came back clean, and clean is what
stopped either of us looking further.

---

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

### THE REVERSAL — the same leak, one step along, and it was PUBLISHED

Fixing *offered in answer order* is not the same as fixing ORDER. Tailwind's
console was corrected off answer order and then shipped offering its six route
ids **in exact reverse**:

    offered : farne -> benguela -> fix-d -> banc-darguin -> fix-b -> destination
    answer  : farne -> fix-b -> banc-darguin -> fix-d -> benguela -> destination
    offered positions within the answer: 0, 4, 3, 2, 1, 5

`farne` is locked into slot 1 and `destination` is fixed last, so **the four
movable items are precisely reversed**. *Try it backwards* is the second thing
anyone tries after *try it as given*, and it takes the middle from 24
possibilities to 1.

**The assertion that caught the first leak did not catch this one**, because it
asked *is the offered order the answer order* and the answer was no. A guard
that names one bad arrangement blesses every other. Both are now asserted: not
answer order, and not its reverse.

**Open, and Maciej's:** `sort-bins` carries a stricter rule — no two adjacent
items sharing a target — which this console would also fail. It was authored
before that rule existed, so applying it retroactively fails correct work
against a check invented afterwards. Recorded with its arithmetic in the test
rather than enforced. **The reversal itself is being fixed regardless; only the
stricter rule is the open question.**

### And a test that expires

`composite-finale`'s test pinned those six route ids in order, so **publishing
Tailwind turned it red for a content edit rather than a defect**. Same shape as
a parse test naming `dial-select` as an *unknown* type shortly before it became
one. A test that pins content goes red when an author does their job, and the
third time it happens somebody deletes the test instead of reading it. **Assert
the shape an author may not break; leave the ordering to the author.**

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
