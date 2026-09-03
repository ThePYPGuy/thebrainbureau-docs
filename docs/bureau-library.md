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

**Twenty-five of the 73 already span two years** " + D + u" `curriculum_skills.uk` reads
`Y3-4` or `Y5-6` rather than a single year. So *Years 3 to 6* is encoded in WHICH
goal you are looking at, not in a fourth axis on top of it. One bank per goal,
20 questions each: **1,460 questions.**

Count it yourself rather than trusting this number:

    select s.subject, count(k.id) from curriculum_skills k
      join curriculum_strands s on s.id = k.strand_id group by s.subject;

## The spread is the point of the bank

Each bank climbs, so one bank serves a mixed class and covers both ends of a
two-year goal:

    rows 1-7    the fact or procedure, asked directly
    rows 8-14   the same skill inside a word problem or unfamiliar presentation
    rows 15-20  multi-step, reasoning, or combined with an earlier skill

A flat bank would have been a legitimate product " + D + u" quick to check, useless for a
mixed class. Maciej chose the spread.

## THE NAMES ARE NOT YET THE NAMES

`rookie | pro | expert` is the ruling. **`recall | apply | stretch` is what the
code still validates** " + D + u" `DIFFICULTIES` in `lib/banks/csv.ts` is the source of the
exported `Difficulty` type, `isDifficulty` rejects anything else at import, and
`bank_questions_difficulty_check` gates it again in the database.

So the 1,460 rows were written with the OLD words on purpose: a file saying
`rookie` is refused today. The mapping is exactly 1:1 in order, so the sequence
is **rename the code, migrate the existing rows, then convert the CSVs in one
pass** " + D + u" never the other way round, and never after import.

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
  and that is the failure this content actually hit " + D + u" three files in the first
  batch, caught by another agent's validator rather than by the importer.
- `explanation` on every row. It is what a child reads after answering, and it
  should teach the method rather than restate the answer.
- `skill_tag` carries the full curriculum skill id, so a bank ties back to its
  goal without a new join table.

## Answer integrity applies to a bank exactly as it applies to an activity

`docs/answer-integrity.md` is the standard. For banks specifically:

- **Vary where the correct option sits.** Count them; roughly even.
- **The correct option must not be identifiable by SHAPE** " + D + u" not consistently the
  longest, the most precise, the most hedged, the only one with a unit, or in
  comprehension the only one echoing the passage's wording.
- **Distractors are plausible errors**, never filler: the off-by-one, the wrong
  operation, the real childhood misconception.

## OPEN, and Maciej's

**How is a Bureau bank distinguished from a teacher's own?** The importer
hardcodes `visibility: "private"` and status `draft`, and wants an `--owner`. So
importing the 73 as they stand produces **73 private drafts owned by one
teacher** " + D + u" the opposite of a library.

Ownerless-and-public with copy-on-use is MY reading, not his ruling: it matches
the existing `copied_from` column and the copy flow, and it keeps one canonical
version the Bureau can correct later. The alternative " + D + u" seeding a private copy
into each teacher's account " + D + u" gives a teacher outright ownership and means the
Bureau can never fix a bad question.

**Cheap now. Expensive after 1,460 rows exist.**
