# Intel and Clearance

**Status: built 2026-08-24. Thresholds and prices are tunable and deliberately
not final — they are guesses until a real class has earned against them.**

Where it lives: the ladder and its thresholds in [lib/clearance.ts](../lib/clearance.ts)
with tests beside it, the two Intel columns in migration `20260824000009_intel_split.sql`,
and the single award site in [lib/server/intel.ts](../lib/server/intel.ts).

---

## Two numbers, not one

This is the single most important decision in this document, and the one that
is painful to retrofit once children have bought things.

| Field | Meaning | Moves |
|---|---|---|
| `intel_earned` | Lifetime Intel ever earned | Only up |
| `intel_balance` | Spendable Intel | Up on earning, down on spending |

**Clearance is derived from `intel_earned`, never from the balance.** If a
single number drove both, buying an avatar would demote a child from Field
Agent back to Junior Agent, which is obviously wrong and would be discovered
by the first pupil to spend anything.

Both rise together when Intel is awarded. Only the balance falls.

---

## Earning

Applies across all four activity types.

| Event | Effect |
|---|---|
| Correct on first attempt | Full award |
| Correct on a later attempt | Reduced award |
| Hint used | Small penalty against that task's award |
| Phase or activity completed | Completion bonus |
| No wrong answers anywhere in the activity | Clean-sheet bonus |

Current per-activity values live in the activity file's `intel` block. They
are placeholders carried from the v0.4 worked example and should be set for
real only after watching one class earn actual numbers.

**Hints cost 25**, deducted from that phase's base award and floored at
zero. `hints_used` is written by `/api/hint`, which serves the text and
records the charge in one call -- the hint does not ship with the task, since
a penalty avoidable by opening devtools is not a penalty.

Three deliberate limits on what a hint costs:

- **Less than a wrong guess.** A wrong answer drops the award by 75, a hint by
  25. A stuck child should ask rather than guess, and the prices have to say
  so or the mission teaches the opposite.
- **Not the bonuses.** Completion and clean-sheet are for finishing and for
  accuracy. Asking for help is neither a failure to finish nor a wrong answer,
  so it does not touch either.
- **Charged once.** Reloading, or clicking twice, returns the text already
  paid for without charging again.

### Repeatable activities

Agent Training can be replayed. Only the **first completion** of a given bank
credits Intel; replays are practice and show a score without paying again.
Otherwise the fastest route to a high total is replaying the easiest quiz,
which makes the number meaningless.

---

## Clearance ladder

Fixed by the product overview: Junior Agent → Field Agent → Senior Analyst →
beyond. The rank names are settled; every threshold is still a guess.

Six ranks, three stars each: eighteen rungs.

| Rank | 1 star | 2 stars | 3 stars |
|---|---|---|---|
| Junior Agent | 0 | 600 | 1,500 |
| Field Agent | 3,500 | 5,500 | 8,000 |
| Senior Analyst | 11,000 | 15,000 | 20,000 |
| Special Agent | 26,000 | 33,000 | 42,000 |
| Chief of Station | 53,000 | 66,000 | 82,000 |
| Director | 100,000 | 130,000 | 170,000 |

**Why stars, and why three on every rank.** Six ranks alone was a ladder a
keen child climbed out of in under two years, on a platform meant to last the
whole of primary. Sub-steps are needed most at the *bottom*, where a Y3 in
their first term needs a win every couple of sessions to stay interested, so
escalating star counts -- fewest at the bottom, most at the top -- would have
put the extra granularity where nobody is. A uniform three also means a child
can read their position without remembering which rank has how many. The
"higher ranks are bigger" feeling comes from the thresholds instead: a
Director star is worth many Field Agent stars.

**Rank carries the hierarchy; stars are progress within it.** Anything
rendering clearance has to keep that order. If stars dominate visually, a
Director on one star reads as *behind* a Field Agent on three, which is
exactly backwards. On the dashboard the rank is display type and the stars sit
under it at body size.

Junior Agent is deliberately cheap: a child's first ever session should pay
out while they are still in it rather than once at the end, so the second star
lands partway through a first mission and a clean run finishes the rank. Field
Agent is then something to come back for.

After that, paced against roughly 1,000 Intel a week for a keen child: a
promotion every two to six weeks, with Director on three stars arriving around
four and a half years in -- a Y6 legend rather than a Y4 formality. A child doing much less finishes primary somewhere around Senior
Analyst, which should feel like a respectable place to stop rather than a
failure to finish.

For scale: a clean run of Zero Hour earns about 1,550, and a first completed
Agent Training run between 500 and 1,500.

Verified against real play-throughs rather than only unit tests: a new agent
completing Zero Hour earns their second star on the final answer, once; and an
agent parked one step below a threshold is promoted by the answer that crosses
it and by no award after.

### Rules

- **Stored, not computed.** The level is written to the agent record when it
  is reached. Changing a threshold later must never silently demote a child
  who has already been promoted — the existing schema comment says exactly
  this and it stays true.
- **Never demotes.** Nothing lowers a clearance level once earned. Raising the
  thresholds to add stars put this to work immediately: an agent promoted
  under the old numbers can now sit above what their Intel would buy, and they
  keep the rank. The progress bar measures from the rung they hold, not from
  their total, so it never offers a promotion they already have.
- **Old ids still resolve.** A stored `field-agent` from before stars reads as
  that rank's first star rather than as unknown. A migration rewrites them,
  but a row it missed must not fall through and drop a child to the bottom.
- **Thresholds are configuration, not a migration.** They will be tuned after
  real classroom data and must be changeable without a schema change.
- **Promotion is an event, not a state.** It should be visible when it happens
  — a debrief moment — rather than only noticed later in a dossier.
- **One award site.** Every grant of Intel goes through `awardIntel()`. That is
  what makes the promotion check impossible to forget when a fifth activity
  type is added; a second place that writes `intel_earned` would silently stop
  promoting anyone.

---

## Spending

`intel_balance` buys upgrades. The shop is a placeholder in Phase 1; the
economy is designed here so the placeholder does not constrain it.

Principles:

- Spending never affects clearance or lifetime Intel.
- Nothing purchasable may confer an advantage in an activity. Cosmetic only —
  avatars, callsign styling, dossier trim. The moment Intel buys a hint or a
  retry, the currency is pay-to-win against children who find the work harder.
- Prices are configuration.

**OPEN:** whether Agent Training's in-game **access tokens** convert to Intel
at the end of a session, or are purely in-round and Intel is awarded
separately. Recommend the latter — tokens are a mechanic inside one game,
Intel is the platform's currency, and conflating them makes every future mode
an economy negotiation.
