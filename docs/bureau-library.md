# The Bureau Library

Maciej, 3 Sep, and this is the whole of what he has ruled:

> *"Create question banks (20 questions per bank) for each learning goal in our
> curriculum for Maths, Science and Literacy. Years 3 to 6. This will be the
> Bureau Library."*

then, asked whether a bank is one difficulty or a spread: **"a spread."**
Then: **"difficulty levels - rookie, pro, expert"**.

Everything below that is not in those three quotes is somebody's inference, and
is marked as such.

## It is 73 banks, not 292, and the data settled that

    mathematics       33 goals
    science           20
    english-literacy  20

**Twenty-five of the 73 already span two years** — `curriculum_skills.uk` reads
`Y3-4` or `Y5-6` rather than a single year. So *Years 3 to 6* is encoded in WHICH
goal you are looking at, not in a fourth axis on top of it. One bank per goal,
20 questions each: 1,460 questions. **Two were dropped on 3 Sep — see below —
leaving 71 banks and 1,420 questions.**

Count it yourself rather than trusting this number:

    select s.subject, count(k.id) from curriculum_skills k
      join curriculum_strands s on s.id = k.strand_id group by s.subject;

## The spread is the point of the bank

Each bank climbs, so one bank serves a mixed class and covers both ends of a
two-year goal:

    rows 1-7    the fact or procedure, asked directly
    rows 8-14   the same skill inside a word problem or unfamiliar presentation
    rows 15-20  multi-step, reasoning, or combined with an earlier skill

A flat bank would have been a legitimate product — quick to check, useless for a
mixed class. Maciej chose the spread.

## THE NAMES ARE NOT YET THE NAMES

`rookie | pro | expert` is the ruling. **`recall | apply | stretch` is what the
code still validates** — `DIFFICULTIES` in `lib/banks/csv.ts` is the source of the
exported `Difficulty` type, `isDifficulty` rejects anything else at import, and
`bank_questions_difficulty_check` gates it again in the database.

So the 1,460 rows were written with the OLD words on purpose: a file saying
`rookie` is refused today. The mapping is exactly 1:1 in order, so the sequence
is **rename the code, migrate the existing rows, then convert the CSVs in one
pass** — never the other way round, and never after import.

## A file is an answer key

`scripts/import-bank-csv.ts` says it in its own header and means it: the CSV
carries a `correct` column for every question, so the script reads it into
memory, uses it, and writes it nowhere. Not to the bucket, not to a temp file.
**The storage bucket is teachers-write, players-read, so a retained upload is an
answer key every player can read.**

That constraint is about the BUCKET. These files live in the repo alongside
activity content, which already carries answers.

## The format, and what an import will refuse

    type,prompt,option_1..option_6,correct,tolerance,explanation,difficulty,skill_tag

- 21 lines per file: a header and exactly 20 questions.
- 13 fields per row. **A comma inside an unquoted field silently becomes a 14th**,
  and that is the failure this content actually hit — three files in the first
  batch, caught by another agent's validator rather than by the importer.
- `explanation` on every row. It is what a child reads after answering, and it
  should teach the method rather than restate the answer.
- `skill_tag` carries the full curriculum skill id, so a bank ties back to its
  goal without a new join table.

## Answer integrity applies to a bank exactly as it applies to an activity

`docs/answer-integrity.md` is the standard. For banks specifically:

- **Vary where the correct option sits.** Count them; roughly even.
- **The correct option must not be identifiable by SHAPE** — not consistently the
  longest, the most precise, the most hedged, the only one with a unit, or in
  comprehension the only one echoing the passage's wording.
- **Distractors are plausible errors**, never filler: the off-by-one, the wrong
  operation, the real childhood misconception.

**`bank:check` IS NOW TWO-SIDED** (`7884bfb`), proving all four content-blind
strategies in both directions — "always the longest" and "never the longest"
both flag, and a control where the key is independent of length stays clean.
The null-model fix is in `lib/banks/guessability.ts` with a test asserting z
near zero on 400 questions where two options always tie for shortest, so the
tie trap that nearly reached Maciej as a false defect cannot recur silently.

## THE LENGTH TELL, which every structural check passed

Written down because it is the most useful thing this job produced, and because
nothing in the pipeline would have caught it.

**The correct option was systematically the longest one.** Measured across the
first draft, against what chance actually predicts — not 25%, but the sum of 1/n
per question, since the banks mix 3- and 4-option items:

    science            112 of 263   chance 66     +70%
    english-literacy   111 of 285   chance 71     +56%
    mathematics         40 of 225   chance 57     -29%

**A child who never read the question and always picked the longest option
scored around 60%** on the worst files. `reading-fluency.2` was 10 of 16.

**Every one of those files passed every structural check** — 13 fields, 21 lines,
explanations present, correct-answer positions evenly spread. The importer would
have taken them. **A bank can be structurally perfect and still measure the wrong
thing**, and no schema constraint will ever find that.

**Why it happened, which is the part worth keeping.** It was not carelessness. A
correct scientific statement needs its qualifying clause where a misconception
can be blunt; a correct inference needs its evidence where a wrong one can be
asserted flatly. **The length follows the truth unless somebody stops it.** Maths
was clean throughout because its answers are numbers.

**The fix was to lengthen the DISTRACTORS, never to trim the correct answer** —
the precision is the thing worth keeping, and a terse wrong option looks wrong
before a child has thought about it. In grammar the same move produced better
questions outright: an error of SUBSTITUTION rather than omission — a comma in
the wrong place rather than a comma missing — costs the same characters and is
the mistake a child actually makes.

After the fix, measured the same way: literacy +3%, science +4%, overall -5%
against chance, and no single file materially above it.

### MY OWN NUMBER FLATTERED THE RESULT, AND A SECOND TOOL SAID SO

`npm run bank:check` — Website Infrastructure's, built independently — measures
the same content and reports:

    ALL   854 questions   chance 25.1%   longest 30.0%   shortest 18.1%
                          first 25.8%    last 25.1%      z=3.2

**Trust that one over mine.** The difference is the treatment of TIES: I counted
a question only when the correct option was UNIQUELY longest, so a two-way tie
scored nothing. `bank:check` counts ties at 1/n. **A child facing a two-way tie
for longest picks one of two, not one of four** — so my stricter filter reports
less signal than a child actually experiences, and my "-5% against chance" is
the flattering reading of the same files.

The honest figure is **30% against a chance of 25%**, down from a first draft
where the worst files paid 60%.

**It does not flag, and that is a threshold rather than an absence.** The tool
flags at five points and this is 4.9. Read `z=3.2` alongside it: the effect is
real and small, not absent.

**And the signal I could not see at all is SHORTEST at 18.1%** — well below
chance, in all three subjects, including maths where the longest signal is weak.
That is the same bias from the other side, and it says the correct answers run
long GENERALLY rather than only where they cross a threshold. My tool asked one
question and so could only ever give one answer.

## NINE FILES WERE BROKEN AND MY VALIDATOR PASSED ALL 73

Worth writing down because the validator was mine and it was confident.

    accepted answer listed twice     "the Earth|Earth|the earth|earth"   4 files
    the same prompt on two rows      "Which spelling is correct?" x5     2 files
    the key matches two options      differing only in capitalisation    1 file
    the key collides with a POSITION "4" when option 1 is the text "4"   2 files

**It checked that the correct answer appears among the options, and that the
options are not duplicated.** It never checked the ACCEPTED-ANSWER LIST for
duplicates, never that a key matches two options, never that two rows share a
prompt — and it could not have conceived of the last one.

**The positional collision is the one to carry into the format documentation.**
A key of `4` is unreadable when an option's TEXT is `4`, because nothing can
tell a position from an answer. Neither row is wrong on its own; **the collision
is fatal and reading the file would never show it.** Fixed by moving the
coordinate off the position range — `(4,7)` became `(8,7)` — and by swapping the
symmetry answer from `A` to `T`, rather than by relabelling anything.

**bank:check reports only the FIRST fault per file**, so fixing exactly what it
names leaves the rest: it named four case-duplicate answers and a generic pass
found the rest. And it refuses to measure a file it cannot parse, which is
right — a quality report over whichever files happened to parse is how a bad set
gets a clean bill of health.


## ALIGNED TO THE TWO MODES THAT EXIST — 3 Sep

Maciej: *"we only want questions that suit the platform we currently have. The
question banks are only playable in two modes so need to be aligned to them."*

**The TYPES were never the problem.** `lib/banks/question-types.ts` already
settles them and says why — *every type must work in every mode, which is why
ordering and matching are absent* — and all 1,460 questions used the three
allowed types.

**The problem was length, and Signal Check is what exposes it.** Live,
whole-class, on a projector, with a hard window of 10-60s (`MIN_WINDOW_MS`,
`MAX_WINDOW_MS`) that must cover reading the prompt, reading the options,
thinking and answering. Solo Practice is self-paced and forgives it. **The
tightest mode binds.**

    before          median words on screen   over 60
    mathematics             20                  9 of 660
    science                 46                124 of 400
    english-literacy        45                121 of 400

**242 questions rewritten across ~40 banks; two banks deleted.** Every row is
now inside the budget. Explanations were NOT shortened — they appear at the
reveal, outside the window — and several were enriched to carry teaching a
shortened prompt gave up.

### The result, measured two ways that agree

    subject             n  |  LONGEST act/exp    z  |  SHORTEST act/exp    z
    english-literacy  287  |    61 / 62.2     -0.2  |    30 / 58.8     -4.3
    mathematics       225  |    38 / 34.4     +0.7  |    15 / 29.0     -3.0
    science           310  |    55 / 69.8     -2.0  |    48 / 64.6     -2.4
    ALL               822  |   154 / 166.4    -1.1  |    93 / 152.3    -5.6

`npm run bank:check` reports the same figures independently and passes the set.

**LONGEST IS CLOSED** — on chance in every subject, from +70% in science and
+56% in literacy.

**SHORTEST IS LOW, AND IT IS LEFT OPEN DELIBERATELY.** The correct answer is
rarely the shortest option. z=-5.6, consistent across all three subjects, and
**pre-existing** — it is the *"correct answers run long generally"* signal
Website Infrastructure named at the outset, not something the rewrite caused.

**The price of exploiting it is smaller than my own first estimate, and the
correction is worth recording because it changes which read is safe to quote.**
My number — eliminating the shortest lifts a child from 25% to about 30%,
roughly 4.6 points — came from `1/n`. WI's tool, `lib/banks/guessability.ts`,
measures **2.3 points, about 27.4%**, because avoiding a two-way tie for
shortest leaves `k-2` options rather than `k-1`, and ties cluster at the short
end — the exact mechanism behind the null-model warning below, arriving a
second time from a different angle. **Both are below the five-point actionable
gate; WI's is further below it.** The decision not to commission a fifth pass
stands, and stands more firmly on the accurate number.

**No fifth pass was commissioned for it.** Four have been spent, it is under the
threshold, and a pass aimed at lengthening correct answers is precisely how the
LONGEST tell returns. It is recorded with its arithmetic so the next person
decides with the number in front of them rather than rediscovering it.

**Two defects were caught by the writers rather than by any check.** One found
its own first pass at 3.1% longest — the inverted tell — and corrected back
toward chance. Another found a pre-existing SHORTEST tell in
`materials-and-their-properties.2` where the correct answer was shortest in 9 of
16 rows and *"always pick the shortest"* scored about 58%; `bank:check` prints
shortest and has never flagged on it, so that file would have shipped. It also
found and removed a mains-electricity question, which the safety rule forbids.

## WHERE THE CURRICULUM ITSELF IS THE PROBLEM

Six goals came back flagged by the people writing against them. None is a defect
in the banks; all are worth a decision.

**Two goals have no year to sit in.** Their `uk` column does not open with a
year band at all: it opens with a sentence saying the topic is not UK primary
content.

- `science.environmental-and-seasonal-change.1` — its `uk` column is not a year
  band at all, it is a note saying **this is not a KS2 topic**: KS1 content
  (Y1, age 5-6) folded into Y6 Living Things and Evolution. The bank was written
  at KS2 pitch — day length, axial tilt, human impact — but **the goal cannot be
  placed in a year group.** Merge it into Living Things/Habitats, or keep it
  standalone with a year assigned by hand.
- `mathematics.probability-where-applicable-at-primary.1` — *"Not a named UK
  primary strand"*. Written to likelihood vocabulary and simple outcome sets.
  Tagged Y3-Y6 for now, which is honest rather than right.

**And one goal is PLACED but is not a curriculum line.**
`mathematics.algebra-at-primary-level.1` opens `Y3–Y4:`, so every parser places
it correctly — but no standalone Y3/Y4 national curriculum statement exists
behind it. It lives inside counting in multiples: a scheme-level construct
wearing a year band.

**I first wrote this up as a third UNPLACED goal. That was wrong**, and Website
Infrastructure's parser contradicted it — two goals fail to open with a year,
not three. The correction is worth more than the count: **a goal can carry a
perfectly good year and still have nothing underneath it**, and a check that
only asks *can this be placed in a year* will pass it every time.

**One strand is misnamed.** `science.earth-and-space.1` **is not about space** —
it is the Y3 Rocks unit: rocks, fossils, soils. The description is accurate; the
STRAND NAME is misleading, and a teacher browsing *Earth and Space* for Y3 will
not expect soil permeability.

**One goal is two goals.** `mathematics.multiplication-and-division.3` — the `uk`
column says *long multiplication/division* while the `skill` says *multi-step
problems using all four operations*. Those are different things. Covered by
putting the algorithm in recall/apply and multi-step work in stretch, but they
want splitting if per-goal reporting is ever wanted.

**One goal carries a whole strand.** `mathematics.ratio-and-proportion.1` has to
hold ratio notation, sharing, scaling recipes, scale factors, maps and unit-rate
comparison in twenty questions. A candidate for splitting.

## TWO GOALS THIS FORMAT CANNOT HONESTLY ASSESS

Said plainly here so nobody discovers it from a disappointing classroom.

**Reading Fluency — DROPPED 3 Sep on Maciej's instruction.** Fluency is accuracy,
pace and expression in the act of reading ALOUD, and a written multiple-choice
question observes none of the three. The two banks tested *knowledge about*
fluent reading — where punctuation asks for a pause, which word carries the
stress. Real and teachable, but **a child could score full marks and still read
haltingly, and a fluent reader who was never taught the vocabulary scored
badly.** The real measure is a running record or a timed read-aloud, which the
terminal cannot capture, so the banks were removed rather than shipped under a
name they could not honour.

**So COVERAGE IS 71 OF 73 GOALS, deliberately.** Verified 3 Sep: exactly
`english-literacy.reading-fluency.1` and `.2` have no bank, and no bank names a
skill the curriculum does not have. **Anyone auditing coverage will find the
gap, and it is not an oversight** — it is written here so nobody fills it back
in by writing twenty more questions that measure the wrong thing.

**The two goals stay in `curriculum_skills`.** They are real curriculum content
that primary schools genuinely teach; it is this FORMAT that cannot observe
them, not the curriculum that is wrong. Deleting the goals to make the coverage
number tidy would hide a limitation of the platform inside a claim about the
curriculum. Fluency belongs to a running record or a timed read-aloud, and if
the platform ever gains a way to capture one, the goals are already there
waiting.

**Writing: Composition and Structure** genuinely assesses paragraph boundaries,
ordering, topic sentences, register, precis and cohesion. It **cannot** assess
whether a child can PRODUCE a structured piece — sustaining a shape over 400
words, keeping a voice, knowing when a draft has lost the thread. These banks
measure editorial judgement about someone else's writing. That is a real
component; it is not the composition skill itself.

## The import run

    npm run import:library -- <dir> --dry-run
    npm run import:library -- <dir> --status published --visibility public

**A permanent link now exists, not just the title text — `skill_id`,
migration 45, Maciej's ruling.** Every bank row was matched to its curriculum
skill by joining on the title, because nothing else carried the id through the
import. That worked, but rested on titles staying unique — and **they already
were not**: `"Copy Source (copy)"` is shared by 4 rows, from double-copying a
bank. The title join was correct for the 71 Bureau banks only because none of
them happened to collide with a copy, not because titles are actually unique.
`skill_id` closes that permanently: a foreign key to `curriculum_skills`,
restricting rather than nulling on delete, so removing a skill while a
published bank teaches it fails loudly instead of quietly cutting the link.

**It proved itself on first use.** A coverage query — which curriculum goals
have no bank — is now one query instead of a diff someone has to notice. Run
the day it landed: 71 linked, 0 mislinked, and exactly the 2 goals with no bank
are `english-literacy.reading-fluency.1` and `.2` — the two dropped on
purpose. Verified independently against the database, not from the migration's
own report.

**Titles are not chosen, they are derived.** Each filename IS a curriculum skill
id — `mathematics.geometry-properties-of-shapes.2.csv` is the skill
`mathematics.geometry-properties-of-shapes.2` — so the title comes from
`curriculum_skills.skill` and the subject from its strand. Nothing is invented
and nothing is typed 73 times. **A file naming a skill the curriculum does not
have is fatal rather than imported under its filename**, because a bank tagged
with a skill nobody can search for is a bank nobody finds.

**Everything is checked before anything is written.** Seventy-three files that
fail on the fortieth leave thirty-nine banks in a library meant to be complete,
with no obvious way to say which thirty-nine. A re-import is refused rather than
skipped, for the same reason: silently skipping what exists is how half a
library gets imported twice and nobody can say which half.

### THE YEAR IS IN THE DATABASE, BUT NOT IN A COLUMN

`curriculum_age_bands` holds Years 3-6 and **nothing links a skill to a band**,
so there is no year column to read. The year is inside `curriculum_skills.uk`,
which is prose that begins with it:

    Y3: to 1,000
    Y4: to 10,000; round to nearest 10/100/1,000
    Y3-4: apply growing knowledge of root words, prefixes and suffixes

**The trailing number in a filename is NOT the year** — `multiplication-and-division`
runs `.1` to `.5` across four year groups. It is a sequence within the strand.

**And a bare `Y(\d)` regex over that column is wrong.** It reads *"(Y1, age
5-6)"* out of the prose note on `science.environmental-and-seasonal-change.1`
and produces a `Y1-Y6` span. **A mention of a year is not a year band.** Clamp
to Y3-Y6, and flag the goals that yield nothing rather than defaulting them
silently — `probability-where-applicable-at-primary.1` has no year at all,
because its `uk` column says *"Not a named UK primary strand"*.

## OPEN, and Maciej's

**How is a Bureau bank distinguished from a teacher's own?** The importer
hardcodes `visibility: "private"` and status `draft`, and wants an `--owner`. So
importing the 73 as they stand produces **73 private drafts owned by one
teacher** — the opposite of a library.

Ownerless-and-public with copy-on-use is MY reading, not his ruling: it matches
the existing `copied_from` column and the copy flow, and it keeps one canonical
version the Bureau can correct later. The alternative — seeding a private copy
into each teacher's account — gives a teacher outright ownership and means the
Bureau can never fix a bad question.

**Cheap now. Expensive after 1,460 rows exist.**
