# Question Banks and Game Modes

**Status: structure decided 2026-08-24, modes revised 2026-08-31. Not yet
implemented — what shipped conflates all three concepts below into one row.**
Verified 2026-08-31: `training_sims` exists; `question_banks`, `bank_questions`,
`training_sessions`, `runs` and `responses` do not.

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
question_banks        title, subject, owner, year_group, tags, status, visibility
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

**The two `year_group` columns mean different things.** On the bank it is the
year the questions were written for: authoring metadata, for search and
filtering. On the session it is the year group actually playing, and it is the
only one the statistics read — sessions carry guests with no account, and every
response needs a year group regardless of who gave it.

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
- **Realtime** transport for scores, attacks and the shared leaderboard.
- **Host controls** — start, pause, skip, kick, end.
- **Reconnection**, because school wifi drops and iPads die mid-round. A child
  who reconnects must return to their score and their tokens.
- **Late joiners**, and a decision on whether they are allowed in.
- **Display-name moderation**, since names appear on a shared screen.

The async modes forgive all of this; synchronous play forgives none of it,
which is why Signal Check is a larger build than its description suggests —
and why it is worth building as the mode whose job is to prove the transport.

**Keep it the thinnest thing that proves the transport** (decided 2026-08-31):
one question to everyone at once, a fixed answer window, no speed bonus and no
individual leaderboard. Every feature it does not have is one that cannot
confuse a failure of the plumbing for a failure of the game.

---

## Authoring

Teachers build banks themselves — that was the reason the mode exists.

**Built:** a manual editor. Add questions, two to six options, mark the correct
one, add an explanation shown on the review screen. Publish to your classes.

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
