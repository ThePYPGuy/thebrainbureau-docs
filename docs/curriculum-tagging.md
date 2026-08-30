# Curriculum Tagging and Search

**Status: built 2026-08-25.** The crosswalk is imported, tasks are tagged, and
labels resolve into the reading teacher's framework.

Where it lives: reference tables from migration `20260825000013_curriculum.sql`,
loaded by [scripts/import-curriculum.ts](../scripts/import-curriculum.ts);
resolution rules in [lib/curriculum.ts](../lib/curriculum.ts) with tests
beside them; the reading side in
[lib/server/curriculum.ts](../lib/server/curriculum.ts).

Source: `Documents\The Brain Bureau\TBB curriculum crosswalk` —
`curriculum-crosswalk.md` (human) and `curriculum-crosswalk.json` (machine).

---

## What the crosswalk gives us

| | |
|---|---|
| Subjects | Mathematics, English Literacy, Science |
| Strands | 32 (11 maths, 10 English, 11 science) |
| Skills | **70** — each a plain-language description |
| Frameworks | UK NC, Common Core, Cambridge Primary, IB PYP, ACARA |
| Age bands | 7–8, 8–9, 9–10, 10–11 |
| Disagreement flags | 24 skills marked `check` |

Each skill already carries its own mapping into all five frameworks. That is
the whole asset, and it is worth more than it looks.

---

## The principle: tag once, resolve five ways

An author tags a task with **one skill id**. They never touch a framework
code. The crosswalk resolves that skill into whatever framework the *reading*
teacher uses.

```
task ──tagged with──> skill ──crosswalk──> UK: "Y6: to 10,000,000; negative numbers in context"
                                           Common Core: "Grade 6: 6.NS.C"
                                           Cambridge: "Stage 6: Number — place value..."
                                           IB PYP: "Phase 5: Number — integers"
                                           ACARA: "Y6: Number — negative numbers"
```

The alternative — asking an author to enter five framework references per task
— is five times the work, guarantees inconsistency, and puts a UK primary
teacher in the position of looking up ACARA codes. This way the mapping is
made once, by whoever maintains the crosswalk, and every activity inherits it.

---

## Age bands, not year groups

Activities are tagged with an **age band**, and the year label is rendered in
the reader's own framework:

| Age | UK | Common Core | Cambridge | IB PYP | ACARA |
|---|---|---|---|---|---|
| 7–8 | Year 3 | Grade 2–3 | Stage 3 | Phase 3 | Year 3 |
| 8–9 | Year 4 | Grade 3–4 | Stage 4 | Phase 3–4 | Year 4 |
| 9–10 | Year 5 | Grade 4–5 | Stage 5 | Phase 4 | Year 5 |
| 10–11 | Year 6 | Grade 5–6 | Stage 6 | Phase 5 | Year 6 |

So Zero Hour is tagged `10-11`, and a British teacher sees "Year 6" while a
teacher in Melbourne sees "Year 6" and one in Boston sees "Grade 5–6" —
without anyone re-tagging anything. Activities currently carry a hardcoded
`yearGroups: ["Y6"]`, which only works for one country.

---

## Alignment status — the 22 `check` flags

Those flags are a feature, not a caveat. They mark skills that are core
content in some frameworks and absent or later in others. Negative numbers,
percentages, ratio, pie charts and the mean are all UK Year 6 content and all
Grade 6 in Common Core — a year later.

The library should say so, per framework, on every activity:

- **Core** — curriculum-required at this age in your framework
- **Extension** — taught later in your framework; usable, but ahead
- **Not in framework** — e.g. probability, which has no UK KS2 presence at all

A US teacher who buys a percentages Operation for Grade 5 and discovers it is
Grade 6 content will ask for a refund. Showing alignment honestly at the point
of browsing turns that into a considered choice, and it is a genuine
differentiator — none of Blooket, Kahoot or Gimkit does it.

---

## Search

Teachers filter by the four things you asked for:

| Filter | Comes from |
|---|---|
| Subject | Strand's subject |
| Learning goal | Skill text, free-text matched |
| Title | Activity title and brief |
| Grade level | Age band, labelled in the teacher's framework |

Plus, falling out of the same data: activity type, alignment status, and
duration. A teacher's framework is a profile setting so labels are right
everywhere without re-selecting.

---

## Skill ids — done

Every skill now carries a stable id of the form `<subject>.<strand>.<n>`, for
example `mathematics.number-and-place-value.4`. All 70 are assigned and
unique, in both the JSON and the Markdown table.

**The rule that keeps them stable: numbers are append-only.** A skill added
later takes the next unused number for its strand even if it belongs
logically in the middle, and a deleted skill's number is never reused.
Renumbering would silently repoint every task tagged against the old number —
no error, just wrong curriculum labels. The rule is recorded in the JSON
itself under `_idScheme` so it survives whoever edits it next.

Content tags against ids and never against wording, so a description can be
reworded freely.

## What was built, and what it does not do

Verified against all five frameworks: one age band tag reads as "Year 6" for a
UK teacher and "Grade 5–6" for a Common Core one, with nothing re-tagged. The
teacher dashboard shows each activity's year and strands, on locked titles as
well as owned ones — curriculum fit is most of the decision to buy something,
and hiding it until after purchase is how refunds happen.

**The three-state alignment was reduced to two and a half, on purpose.** The
proposal promised Core / Extension / Not in framework, where *Extension* meant
"taught later in your framework". Saying that truthfully needs a structured
year per framework, and the crosswalk carries prose — `"Grade 6: 6.NS.C"`.
Parsing that to decide "later" would be wrong often enough to mislead exactly
the teacher it is meant to protect. So what ships is:

- **Core** — mapped, no recorded disagreement about when it is taught
- **Timing varies** — mapped, but frameworks disagree with each other
- **Not in your framework** — no mapping at all

Note the wording of the middle one. The `check` flag means the frameworks
disagree *with each other*, not that the reader's is the odd one out. Telling
a UK teacher to "check the year" on negative numbers — which sit exactly where
they expect, in Year 6 — would be both wrong and noisy. Getting a true
"this is a year ahead for you" needs structured years, and that is a crosswalk
change, not a code change.

**Three gaps in the crosswalk, since closed.** Found while tagging real
content, and there were three rather than the two first reported — square
numbers was the one nobody counted:

- **Factors, multiples and primes** — `multiplication-and-division.4`
- **Square numbers** — `multiplication-and-division.5`
- **Order of operations** — `algebra-at-primary-level.3`

All three carry `check: true`: the UK column is authoritative and the others
are best-effort rather than sourced.

**Two of them are Year 5, not Year 6.** Factors/multiples/primes and square
numbers sit in Y5 in the National Curriculum; only order of operations is Y6.
They were recorded as "three Y6 blocks" for several days. It matters for
pitching an activity, not just for the file — a Y6 class has already met two
of them.

Until they existed those locks were tagged to the nearest honest match, which
is the failure worth remembering: a near-miss tag is not a smaller version of
the right tag. It makes the Operation answer a search for something it does
not teach, and one attempt had Lock 02's long multiplication tagged as
addition.

**Coverage still to decide.** Global Intel Cards is economics and geography
tagged against its maths; Field Ops will be cross-curricular. Either the
crosswalk grows a Humanities section, or those activities carry a
plain-language tag with no framework mapping.

---

## Still to do

✅ **The crosswalk lives in the repo** at `content/curriculum/` and imports
into tables — strands, skills with their five framework columns, age bands,
and a join table from task to skill. Same pattern as activities: the file is
the authoring format, the tables are the runtime.

✅ **Tags belong on tasks, not just activities**, and that is where they are.
The per-task view can therefore say which lock teaches what, once it is built.

**Search is not built.** The data supports it — subject, skill text, title and
age band are all queryable — but there is no filter UI yet.

**A teacher cannot change their framework.** The column exists and defaults to
`uk`; nothing in the interface sets it.

**No per-activity page.** The dashboard row shows year and strands; the full
skill wording has nowhere to live yet.

**Coverage gaps to decide on.** The crosswalk covers Maths, English and
Science. Global Intel Cards is economics and geography; Field Ops will be
cross-curricular. Either the crosswalk grows a Humanities section, or those
activities carry a plain-language skill tag with no framework mapping and are
searchable but not framework-aligned.
