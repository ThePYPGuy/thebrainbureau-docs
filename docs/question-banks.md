# Question Banks and Game Modes

**Status: structure decided 2026-08-24, modes revised 2026-08-31, built and
merged 2026-08-31 — live in production at `b663ad3`.**
Do not trust that either. `npm run deploy:check -- --prod`, from the main
worktree, is what answers it. The first version of this line named which tables
existed and was false within the afternoon; the second said *not yet merged* and
was false within the hour. A status line about a moving thing is a dated claim,
and this one is dated.

---

## Three things, currently one

The product overview requires that **a bank built for one mode runs in any
future mode without rebuilding the questions**. That is only true if the
questions, the rules and the session are separate.

| Concept | What it is | Owned by |
|---|---|---|
| **Question bank** | Questions and answers. Subject matter, nothing else. | A teacher, or the Bureau |
| **Game mode** | The rules. Mainframe Breach, solo practice, future modes. | Code, not data |
| **Session** | One playing of one bank in one mode, live or solo. | An agent, or a class |

What shipped as `training_sims` is all three at once: it holds questions
*and* `run_length` *and* `intel_per_correct`, so its questions cannot move to
another mode without dragging one mode's settings along.

### Target shape

```
question_banks        title, subject, owner, year_groups, tags, status,
                      visibility, copied_from
  └── bank_questions  question_type, prompt, options, explanation, position,
                      image, option_images, alt_text
        └── keys      the correct answer — server-only, as now

training_sessions     bank + mode + config + class + status + game PIN +
                      year_group
  └── runs            one agent's play of one session
        └── responses what they answered
```

Mode configuration — how many questions a round serves, what a correct answer
pays, whether attacks are enabled — belongs to the **session**, not the bank.

**The two year-group columns mean different things, and differ in cardinality
because of it.** `question_banks.year_groups` is a `text[]` — the years the
questions were written for, authoring metadata for search and filtering, and
plural because a bank is routinely authored for several: all three Bureau quizzes
are, and `training_sims.year_groups` is already an array, so a singular column
would have discarded authored data in a backfill this document requires to lose
nothing. `training_sessions.year_group` is singular — the one year group
actually playing — and it is the only one the statistics read, since sessions
carry guests with no account and every response needs a year group regardless of
who gave it.

This document specified the bank side singular and the implementation deviated,
with the reasoning recorded in `20260831000019_question_banks.sql`. The
distinction the spec was drawing was *meaning*, not cardinality, and it survives.

### Question types — decided 2026-08-31

`question_type` is a column, and each type carries a validation handler and a
rendering contract, so a fourth type later is one handler rather than a
migration and a change to every mode.

- **multiple_choice** — two to six options, one correct, shuffled per run.
- **numeric** — free entry, no options, with an optional tolerance defaulting to
  exact. The gap this closes is maths: *47 × 6* as multiple choice is a
  different task from working it out, because the options do half the thinking
  and can be back-solved. Units are never part of the answer; they belong in the
  prompt.
- **short_text** — free entry for spelling and vocabulary, case-insensitive and
  trimmed, holding a **list** of accepted answers rather than one string. No
  fuzzy matching: silently accepting a misspelling in a spelling test is worse
  than being strict.

**Every type must work in every mode.** A type that functions in only one breaks
the premise of the refactor — which is why these two and not ordering or
matching, both of which need drag interactions and do not fit a projector-paced
answer window.

### The bank is the page, and the modes are actions on it — decided 2026-08-31

A live session is **started from a bank**, not from a separate part of the
product. There is no *Live* section in the navigation.

That is the interface saying what the schema already says. A bank is
mode-agnostic; a session is a bank plus a mode. So the bank gets a page, and each
mode is a button on it:

- **Start live session** — Signal Check, the whole class, now.
- **Play solo** — Solo Practice, async, one child.
- **Assign to a class** — the existing deployment route.

One page, three verbs, and a fourth mode later is a fourth button rather than a
new area of the product. The alternative — a *Live* section beside the library —
splits one bank across two places and has to answer *which bank* all over again.

**Navigation:** *Discover* for banks other teachers have made public, *Bureau
Library* for the official ones, *Create* for your own. Reference for layout and
density is Blooket, which this shape is taken from; `docs/brand.md` carries how
it looks, this file carries why it is arranged this way.

Two things this makes real that are already decided: public banks are searchable
with a **play count** beside each, and *Take a copy* is how a teacher gets an
editable version of someone else's.

### Sharing — decided 2026-08-31

A bank is **public or private**, and nothing between. Private is the owner's
alone. **Public means everyone on the platform**, across schools: one global
pool, not a per-school setting and not a share list. So `visibility` is a flag
on the bank, and there is no join table recording who a bank was shared with —
which is the whole reason to settle this before the migration rather than after.

**Anyone may use a public bank. Only its owner may edit it.** Running a session
from someone else's bank takes no copy — copying is what you do to *change* one.
A teacher who wants a bank they do not own to be different takes a copy, and
that copy is theirs. So a bank has one owner, and needs no permission model
beyond `visibility`.

Because most use is direct rather than copied, plays and difficulty accumulate
on the one bank instead of scattering across forks. A copy does start its own
statistics, so it carries `copied_from` to its source — provenance, and the
option of pooling a lineage later.

**The case that needs care is a bank being used by teachers who do not own it.**
Its owner can still edit it, retire it, or make it private, and none of those
should reach into a lesson somebody else is running. See *Reproducibility*.

Public banks are **searchable**, and the search screen shows a **play count**
beside each, so a teacher can tell a bank that has been used from one that has
only been published.

**Derive that count from the sessions, never keep a running total on the bank.**
A maintained counter drifts the first time a session is deleted or a run dies
halfway, and it drifts *silently* — the stale number still looks like a number.
Same rule as the per-year-group difficulty tracking, and for the same reason.

### Reproducibility — decided 2026-08-31

**A session snapshots the questions it serves, at the moment it starts.** From
then on the session is self-contained: what a class played is what the record
says they played, and nothing the bank's owner does afterwards reaches into it.

That is the whole reason for the snapshot. A bank can be used by teachers who do
not own it, so without one an owner editing a question changes a game already
running, and re-labels answers already given to different wording.

`session_questions` holds the prompt, the options and the order actually served.
`responses` point at the row that was served rather than at the bank's current
one. Each snapshot row keeps `source_question_id`, so difficulty still
aggregates across every session that ever served that question — the snapshot
fixes history without fragmenting the statistics.

**The snapshot must not become the leak.** The served key is snapshotted too,
and it belongs in the same server-only place the bank's keys live, under the
same grant. A copy of the answer in a table the client can read is the one way
this design fails, and it would fail quietly.

Two rules follow:

- **Archive, never hard-delete.** A deleted question orphans every response that
  answered it, and the teacher insight view loses data with nothing to say so.
- **Turning a bank private stops new sessions only.** Sessions already running
  or already finished are untouched — a teacher does not lose the lesson they
  are halfway through because someone else changed their mind.

### Status, and the room for moderation

`status` is a constrained column, the way `skin` is on activities — a value is a
decision, not a string anyone can invent. It carries `draft`, `published` and
`archived` now, and **`pending_review` and `rejected` from the start**, though
nothing sets them yet.

They are there because adding a value to a CHECK constraint later is a
migration, and having them costs nothing. Whether a bank going public should be
reviewed by anyone is undecided; this makes that a change of application logic
rather than a schema change under a live table.

The gate is **playable if and only if `status` is `published`**. Written that
way round, the two unused values are already safe: a bank in a state nothing
handles is unplayable by construction rather than by having been remembered.

### Migration from what exists

The three Bureau quizzes become banks; the solo game becomes the first mode.
Nothing authored is thrown away. `run_length` and `intel_per_correct` move
from the bank to session config, and existing runs point at a session created
retroactively.

---

## Modes

### Solo Practice (built, shipping)

Async, single player, against three simulated rogue nodes. Answer to earn,
crack a node's passphrase to recover Intel it holds. A recovery window every
third question; the passphrase shortlist narrows with each correct answer; one
attempt per node; a window only opens if something was answered correctly in
that round.

Kept as a mode because it is genuinely useful — a child can practise a bank
alone, at home, without the class or the teacher.

### Mainframe Breach (single player, and after Signal Check)

Correct answers earn **access tokens**; tokens buy attacks, defences and
intelligence moves. Originally specified as the live whole-class mode; **Maciej
moved it to single-player on 2026-08-31** so that the multiplayer problem is
solved once, in one place, rather than inside a mode that also has to be
designed. It keeps its token economy and its fiction.

**It is built after Signal Check** (decided 2026-08-31). The two are both modes
under Agent Training and neither replaces the other; Signal Check goes first
because it is the thinner of the two and proves the transport they share.

### Signal Check (Phase 2 — the live one, and the next one built)

Live, whole-class, synchronous, and **the test for the multiplayer environment**
rather than a game that happens to be multiplayer. That is its purpose: if
synchronous play works here it works for any later mode, and if it does not,
only one mode is affected.

Needs, none of which exists yet:

- A **session lifecycle** — lobby, start, rounds, end — with a game PIN.
- **Realtime** transport for the question, the answered count and the reveal.
  **No leaderboard on any surface, and no attacks** — attacks and tokens belong
  to Mainframe Breach, and this list described that mode before Signal Check
  took the live slot.
- **Host controls** — start, pause, skip, kick, end.
- **Reconnection**, because school wifi drops and iPads die mid-round. A child
  who reconnects returns to their score and the current question.
- **Late joiners**, allowed, entering at the next question.
- **Guest nickname moderation at the lobby**, since a nickname is free text and
  reaches a shared screen. There is no display-name entry for agents — they play
  as their codename, and no real name exists anywhere to moderate.

The async modes forgive all of this; synchronous play forgives none of it,
which is why Signal Check is a larger build than its description suggests —
and why it is worth building as the mode whose job is to prove the transport.

**One broadcast into a room of thirty is thirty metered events**, because
Realtime counts each delivery to each client. Measured 2026-08-31 against thirty
real sockets: **96 msg/s at peak**, 3.6 deliveries per player per question.

So the binding limit is **throughput, not monthly volume** — the question this
was first specified to answer, *does a session burn six figures a month*, was
the wrong one. Any future live mode inherits the arithmetic: the ceiling is
per-second and it is reached by one classroom, not by a term of them. And
Realtime does not error at the ceiling. It stops delivering.

**Keep it the thinnest thing that proves the transport** (decided 2026-08-31):
one question to everyone at once, a fixed answer window, no speed bonus and no
individual leaderboard. Every feature it does not have is one that cannot
confuse a failure of the plumbing for a failure of the game.

### The podium — reversed 2026-09-01

**A podium of the top three. The teacher sees every result; the class sees three.**

*This reverses the line above, and the reason it reverses is that the reason has
expired.* "No leaderboard" was never about ranking being wrong — it was about
keeping the mode thin enough that a transport failure could not be mistaken for a
game failure. **The transport is now proven**: 96 msg/s across thirty real
sockets, and a full game played end to end. The condition the absence rested on
is met.

**The split is the whole design.** Every result to the teacher is assessment;
three names to the room is a moment. Thirty ranked names on a projector is a
child seeing themselves last in front of their class every week, which is the
thing that was actually worth avoiding.

**Two things it collides with, both real:**

**Ties are unbreakable as scored.** `engine.ts:562` does `score += 1` on a
correct answer and nothing else — there is no speed component, by an earlier and
separate decision. With thirty children and twenty questions a "top three" will
routinely be a top nine. **Either shared places are the design** — which is
kinder and needs saying out loud — **or speed comes back as a tiebreak**, which
reverses a second decision and changes what the game rewards.

**A guest's name goes on the projector in the largest type in the room.** Display
names are free text, and `20260831000020_training_sessions.sql:207` already flags
moderation as needed for exactly this reason. A podium sharpens it from a lobby
concern to a front-of-class one.

---

## Authoring

Teachers build banks themselves — that was the reason the mode exists.

**Built, for `training_sims`.** The sim editor adds questions, two to six
options, marks the correct one and adds an explanation shown on the review
screen.

**Not carried over to banks yet.** The bank editor built alongside the refactor
handles difficulty tags, the confidence threshold and a question-level image —
but it does not edit questions, so **CSV import is currently the only authoring
path for a bank**. The sim editor holds the model for closing this.

**Take a copy** landed 2026-08-31. A copy is a fork and not a link: new question
rows with new ids, no slug, no `content_hash`, no archived questions, and the
statistics stay with the original by construction rather than by anything
deleting them. `copied_from` is provenance only — nothing reads through it at
runtime.

Still outstanding: **per-option images in the editor**, and the gap is the
interface alone. The columns carry them, the CSV parser reads `option1image` and
`option1alt`, `check:alt` covers them, and editing a question **preserves** the
ones it already has rather than dropping them. So a bank can hold per-option
images that cannot be added or changed on screen; changing one means
re-importing.

Unlike *Take a copy*, nothing in the interface promises otherwise. It is an
absence rather than a broken promise, which is the distinction that kept it from
holding the merge.

**Not built: CSV upload.** Requested 2026-08-31, and the design question is not
the parsing. It is that **the answer key cannot live in the same place as the
questions** — keys are in a table no client-facing role can read, and that rule
holds for every mode. So a CSV either carries the key and is a file that must
never be served back, or it does not and the teacher marks answers after
import.

**Settled 2026-08-31.** The CSV *does* carry the key, in a `correct` column, so
the uploaded file is an answer key for every question in it. It is therefore
parsed in memory and **discarded** — never written to storage, never left on
disk, never attached to the bank for re-download. The bucket policy is teachers
write and players read, so a retained upload is a key every player can read. The
whole file is validated before any insert, so a bad row rejects the file rather
than importing its neighbours and reporting afterwards.

**Re-upload settled 2026-08-31.** Uploading a CSV always creates a **new** bank.
Replacing the questions in an existing one is a separate action, taken against a
bank the teacher picks from a list — never inferred from a filename or a title,
because a wrong inference silently overwrites the wrong bank. Questions the
replacement drops are archived, not deleted, so nothing that has been answered
is destroyed. This is the shape the activity importer already has: match on a
stable identifier, update in place, and never delete-and-recreate —
`npm run test:reimport` exists to prove that last part.

**Not built:** AI-assisted generation from a topic or pasted text, with teacher
review before use. This is in the locked overview and needs a model call, a
cost model, and a review-and-edit step that makes clear the teacher is
accountable for what their class sees. No generated question should ever reach
a child unreviewed.

### Images

**Not built.** A question may carry one image, and each multiple_choice option
may carry one. Teacher-uploaded, held in storage and served from there.

- jpg, png and webp — **rejected by content type, never by file extension**.
- A size cap, starting at 2MB, and a server-side resize to a sensible maximum
  dimension. An unresized phone photo does not arrive in time on school wifi,
  and the question is on a projector in front of a waiting class.
- **Store an image reference, not a raw URL**, so storage can move without
  rewriting every row that points into it.
- Alt text is required on every image, and must not carry the answer — the rule
  and the reason are under *Rules that hold across every mode*.

CSV import does not carry images. They are added afterwards in the editor, and
the import screen says so rather than leaving teachers to work it out.

### Search

**Not built.** Banks list and filter by subject, year group, tags, owner, status
and visibility, with the play count beside each.

A teacher must be able to find a bank they made last term, and to find public
banks made by anyone. Authoring a bank that cannot be found again is most of the
way to not having authored it.

---

## Difficulty

**Not built**, though the raw serve log it rests on is part of the refactor
itself: every serve is a row carrying question, session, year group, correct,
milliseconds and timestamp.

**Derived from those raw rows, never from a running average.** An average cannot
be recomputed when the formula changes, and the formula will change. The same
rule as the play count, for the same reason.

**Per year group, never globally.** A question hard for Y4 and easy for Y6 has
no single difficulty; aggregating the two produces a middle that describes
neither year and never separates back out.

**Across every school, though — and that is why the stats views cross RLS on
purpose.** `bank_question_stats` and `bank_play_counts` run as their owner
rather than as the caller. Supabase's Advisor flags both as *Security Definer
View*, correctly identifying the pattern and unable to see the intent.

Leave them. They return counts, a year group label and a timestamp — no
identity, no free text — and `anon` is revoked on both. The view that returns a
child's typed answer is a different view and it *does* set `security_invoker`.

**Making the Advisor green would break the threshold silently.** Scoped to the
caller, the counts become per-teacher, never reach twenty plays for any year
group, and the rating never lifts. Nothing errors; the editor simply says *not
enough to rate yet* forever, which is indistinguishable from a class that has
not played much. The reasoning is also written into
`20260831000023_insight_views.sql` beside the views themselves.

**And the Advisor does not merely display this — it hands out a written request
to fix it.** Its *Ask Assistant* control generates a prompt reading *"suggest
fixes for the following lint item"*, naming the view. An agent given that will
answer it correctly and destructively, because `security_invoker = true`
genuinely is the right answer to the question as asked, and nothing in the
prompt carries the reason the view is owner-run. That happened within the hour
on 2026-08-31.

The findings **cannot be muted**, so the Advisor reads **2 CRITICAL
permanently**. That is a known state, not a new one. Treat any proposal to make
it green as a change to the difficulty threshold, because that is what it is.

**Guest responses count.** The statistic is about the answer and not the
account, and dropping guests skews the sample toward whoever happened to have a
login.

**History has no year group, and cannot be given one.** The runs that predate
sessions were migrated with a null year group, because nothing recorded one at
the time. They aggregate under *unknown* and sit far below the threshold, so
they show as a count rather than a rating. This is correct rather than a defect
to fix: a year group invented for them would be a guess presented as data, and
per-year-group difficulty simply starts accumulating from the first live session.

### Two kinds of bank, and only one is tagged in detail — decided 2026-08-31

**Bureau banks are tagged in detail. Teacher banks may be, and are never made
to be.**

The two are already distinct in the schema rather than by convention: a Bureau
bank carries a `slug` and a `content_hash`, because it came from a file the repo
tracks. A teacher's carries an `owner_teacher_id` and neither.

**Curriculum tagging is real work, and most teachers will not do it.** Requiring
it would not produce tagged banks; it would produce fewer banks, which is worse
than an untagged one. The Bureau library carries the tagging burden because
being searchable by curriculum is the whole of its value — that is what a
teacher comes to it for, and it is why the Bureau writes it rather than asking
someone else to.

The schema already behaves this way and should stay that way:
`bank_question_skills` is a join table, so absence is the default, and the
importer validates only what is supplied — a typo fails the whole file before
anything is written, while supplying nothing is silent. **A tag that exists is
trustworthy; a missing one costs nothing.**

**The trade, stated so nobody later "fixes" it:** an untagged teacher bank will
not appear in a curriculum search. It stays findable by subject, year group,
title, tags and play count. That is the correct price and requiring tags is not
the remedy.

*Implemented 2026-08-31* (`35272de`): the JSON path reads `difficulty` and
`skills`, which only the CSV importer could before. All three library banks were
re-imported tagged and rated — 66 questions, every one of them. The rule the
policy rests on is now stated where it is enforced: **a missing tag is a
teacher's choice; a wrong one is a lie.**

Teachers tag difficulty when authoring — **recall / apply / stretch**. Live data
overrides that tag only above a confidence threshold, **starting at 20 plays for
that year group**. Below it the editor shows the count instead — *answered 6
times, not enough to rate yet* — rather than adapting on noise.

**The threshold is visible in the editor**, because a system acting on thin data
looks exactly like one acting on good data, and the teacher is the only person
in a position to notice which it is doing.

**For numeric and short_text, log the wrong answer itself**, not only that it
was wrong. Thirty children typing the same wrong value is a misconception worth
putting in front of a teacher. Multiple choice cannot show that; free entry can,
and it is a good part of why those types are worth having.

## Teacher insight

**Not built.** A per-bank view: which questions are answered wrong most often,
per year group, and for numeric and short_text the most common wrong answers.

**This is a teacher's screen, and it is the only place those typed answers are
shown** — never a projector, never a shared surface. See *Nothing a child typed
goes on a shared screen* under the rules below.

Worth shipping on its own, independent of any mode. It surfaces misconceptions
and badly-worded questions, and it is the one thing here that pays a teacher
back for authoring rather than asking something further of them.

---

## Rules that hold across every mode

- **Answer keys never reach the browser.** Keys live in a table no
  client-facing role can read; validation is server-side; the client learns
  only whether it was right and roughly how far off. Already true and tested;
  it stays true for every future mode.
- **Options are shuffled per run**, so replaying a bank cannot be beaten by
  remembering positions.
- **The review screen names the right answers, and only afterwards.** During
  play the same information is the answer key.
- **Nothing a child typed goes on a shared screen.** A `short_text` answer is
  something a child wrote, and a projector puts it in front of the whole room at
  eight metres — where the ones worth showing a teacher are exactly the ones a
  child would least like shown. Decided 2026-08-31, when a live reveal had been
  specified to display the three most common wrong answers.

  Aggregates are safe, because nobody wrote them: how many chose each *authored*
  option, how many answered correctly, and which questions the class got wrong.
  Those are what a shared screen shows.

  **The typed values are still logged and still reach the teacher** — in the
  per-bank insight view, which is a teacher's screen and not the room's. The
  misconception is not lost; it moves to the surface where only one adult reads
  it. That is also why the wrong answer is worth logging at all.
- **Free-entry keys cannot be leak-checked by value.**
  `scripts/test-answer-leak.ts` matches secret strings in the served payload and
  sets `DIGITS_FLOOR = 3`, saying in its own comment that short numeric answers
  are not checked and cannot be — a two-digit answer is indistinguishable from
  any other number on the page. Numeric and short_text keys need a guard that
  asserts **shape**: that no client-facing query selects the keys table and no
  serialised payload carries a key field. Prove it by pointing a client-facing
  query at the keys table and confirming the guard fails.
- **Alt text must not carry the answer.** It reaches the browser as a public
  string like any other, and on option images it is acute: *which shape is a
  hexagon*, with the right option described as *a hexagon*, hands the answer to
  the one group who cannot see the image. Physical detail only, never what makes
  an option correct. The reason is in `CLAUDE.md` under *Alt text can be the
  answer*.
