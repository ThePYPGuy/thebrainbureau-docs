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
question_banks        title, subject, owner, tags, status
  └── bank_questions  prompt, options, explanation, position
        └── keys      the correct answer — server-only, as now

training_sessions     bank + mode + config + class + status + game PIN
  └── runs            one agent's play of one session
        └── responses what they answered
```

Mode configuration — how many questions a round serves, what a correct answer
pays, whether attacks are enabled — belongs to the **session**, not the bank.

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

### Mainframe Breach (single player, for now)

Correct answers earn **access tokens**; tokens buy attacks, defences and
intelligence moves. Originally specified as the live whole-class mode; **Maciej
moved it to single-player on 2026-08-31** so that the multiplayer problem is
solved once, in one place, rather than inside a mode that also has to be
designed. It keeps its token economy and its fiction.

### Signal Check (Phase 2 — the live one)

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
import. Decide that before writing a parser. Also unsettled: the column
contract, what happens to a malformed row (reject the file, or import the rest
and report), and whether re-uploading a bank updates it or duplicates it.

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
