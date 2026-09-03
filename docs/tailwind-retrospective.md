# Why Tailwind took so long, and what Encore should do differently

Written 4 Sep at Maciej's request, from three retrospectives — Operation
Builder (who built it), Lock Library, Website Infrastructure — plus direct
verification of every load-bearing claim. Website Designer's had not arrived
when this was written.

**Operation Builder's account is the only first-hand history of the build.**
Theirs was written from git rather than recall, because their context was
compacted mid-build. Lock Library and Website Infrastructure corroborate the
structural findings from other angles and say plainly where they cannot speak.

---

## The shape of the spend

**~28 hours elapsed, ~150 commits, and roughly a fifth to a quarter of it was
second attempts.** Not hard problems — repeats:

    the quad extractor          three times, still carrying a correction
    the console control         twice (spinners allowed an unrepresentable state)
    three console props         regenerated (`--ref` silently overrode `--aspect`)
    `ordering` rendering        effectively twice
    both matching columns       re-authored after the answer was readable across the row
    the board itself            built as a list, rebuilt as hotspots

## Where the money actually went — three categories, none of them difficulty

**1. SEAMS. Two halves, each individually correct, that never met.** This is the
big one and both retrospectives reached it independently. Of the five defects
Maciej found in a single playthrough: a renderer and grader that never met, an
answer key that was the identity mapping of two columns, a control allowing an
unrepresentable state, CSS classes referenced and never written, a scene never
drawn. **Not one was a missing capability.** In Operation Builder's words:
*every expensive hour went on two halves that were each correct.*

**2. ROUTING. Material existed and nothing carried it to the person acting.**
The scene notes — carrying every measurement the rebuild needed — were written
at 13:28 and transcribed into the repo at 02:21 the next day, **thirteen hours
later**, by the session doing the rebuild. Twelve props sat measured, correct
and referenced by nothing for ten hours. `findAnswerInHint` existed on another
branch and had never been run against Tailwind, while one hint handed a child
the answer they had paid Intel for. **The failure is never "material was
missing".**

**3. INSTRUMENTS. Operation Builder had no working browser for an entire
session** — the pane reported `innerWidth 0`, a fresh tab did not recover it.
**Every fix that evening was verified by somebody else.** That is not a skill
or asset problem and it is cheap to check before starting.

---

## The finding that changes the plan

**THE TWENTY LOCK TYPES HAVE NO ROAD INTO A REAL OPERATION.** Lock Library
found it; verified here three ways before writing it down:

    lib/engine + lib/server/check.ts   import nothing from lib/locks
    components/terminal/               renders no lock, imports nothing from lib/locks
    scripts/import-activity.ts         has no lock support
    tasks_task_type_check              accepts no lock type as a value

Tailwind's live console is `task_type: "console"`, graded by the old engine.
`composite-finale` does nearly the identical job and **nothing in production
calls it.** It is a complete, well-tested, parallel implementation with no
on-ramp.

**So "the locks are ready" is true about the engine and false about the
on-ramp.** Authoring a lock is not slow; there is nowhere to write one, nothing
to import it, and nothing to draw it. That pipeline is new engineering and
costs roughly the same whether Encore uses one lock type or twenty.

**Lock Library's estimate: of what one real published lock instance will cost,
the part twenty ready types shrinks is about a fifth.** The rest is the missing
pipeline, the bespoke config/answer/art every instance needs regardless, and
the human proof-by-playing pass.

---

## The hypothesis, tested rather than agreed with

**"Twenty lock types ready" — keep it, for a reason other than the one
assumed.** A new task type costs **five coordinated edits** — migration CHECK,
the `TaskType` union, a `check.ts` branch, a `Tasks.tsx` case, `TASK_TYPES` in
the importer — with a sixth trap behind it: `import_activity()` carries *two*
column lists, and missing the `on conflict do update set` one makes a column
populate on first import then silently freeze forever while `deploy:check`
stays green. Removing that chain is real value. The validator work also caught
two live bugs — a grid-rounding fault and the offBy oracle leak — that would
otherwise have shipped inside Encore's content too.

**But it does not touch what cost the most, and it may add to it.** Twenty
ready types is twenty *more* halves. Twenty renderers that grade correctly and
draw wrongly is the same night at four times the scale. Nineteen have never
been solved end-to-end by a human with a real answer.

**"Room images ready" is the weaker half.** Generating them was never the
bottleneck — **wiring them was.** Twelve existed, correct and measured, and
reached no page for ten hours. Pre-generating optimises a step that was already
cheap and adds another asset that can sit unreferenced.

---

## Traps that will fire again on Encore as things stand

Website Infrastructure's predictions, verified here:

**The shared-vocabulary trap.** There is no shared task-type constant in
`lib/`. One exists in `scripts/import-activity.ts`, and
`scripts/test-task-types.ts` reads it **out of that file by regex on the source
text**, because it cannot be imported. The literals appear independently in at
least `lib/locks/types.ts`, `lib/engine/types.ts`, `lib/server/check.ts` and
the importer. **Somebody already felt this and built a workaround instead of a
fix.** Any new typed value Encore adds repeats it, and nothing counts the call
sites.

**Freshness across handoffs.** Three instances in one evening — a stale local
`main`, a migration number overtaken, a file count taken mid-rewrite. Structural
to many-worktrees-one-trunk, not to what Encore builds. Fires on any handoff
with a gap greater than zero seconds.

**An internal identifier reaching a column people read.** `curriculum_strands.subject`
went straight into a teacher-facing filter, so teachers would have seen
`english-literacy` beside `English`. Fires again if Encore imports content from
any generated or external source.

---

## The uncomfortable finding

**The most expensive failures had written remedies already in the repo, and
they were not followed.** `CLAUDE.md` carries, from after Prime Directive:
*ask for a playable-but-ugly version at about 30%*, *hand over the mockup
first, as a file, whole*, *answer the design questions in one go*. Tailwind
followed none of them and repeated the failure they were written to prevent.

**So the missing thing is not knowledge. It is a gate.** As advice this has now
failed twice. Operation Builder's conclusion, which I agree with: make it a
precondition — **art cannot be commissioned until a playthrough has happened.**

**And on the checklist question specifically:** a checklist is the right
instrument for *is this complete* and structurally the wrong one for *is this
the thing that was asked for*. Twelve acceptance items passed on a board of the
wrong shape, none of them saying "hotspot" — and a list that had said "hotspot"
would have been written by someone who already knew the answer. **You cannot
lengthen your way out of that.** The instrument for shape is a person using it
early. Maciej found five defects in one playthrough that 569 tests and twelve
acceptance items had all passed over.

---

## What should exist before Encore starts, in order

**1. The lock on-ramp.** Activity-schema slot, importer support, terminal
dispatch. **Nothing else on this list matters until this exists** — Lock
Library's ordering and it is right.

**2. The authoring guard wired INTO that import path.** It already exists, it
already works, and it **already refused this exact fault** — `assertLockAuthoring`
rejected Lock Library's own reference instance on its first run for offering
its route in answer order, which is the identical failure that shipped live in
Tailwind and was caught only by a person looking at a screen. Today it is
invoked from tests and an internal gallery. **The machinery works and guards
nothing real.**

**3. One task-type vocabulary in `lib/`, imported everywhere**, plus a check
that every CHECK-constraint vocabulary has exactly one TypeScript source. This
is the fix for the five-edit chain and it is buildable now.

**4. A single test that, for every authored task type, mounts the renderer,
submits a correct answer, and asserts it grades.** Operation Builder's
suggestion and the highest-leverage item here: one assertion covering the exact
seam that produced the item-5 defect. **Worth more than the other nineteen
types combined.**

**5. The 30% playthrough as a GATE on commissioning art.** Not advice.

**6. The mockup and its measurements transcribed into `docs/` before any build
starts.** Also a gate — Tailwind's notes existed thirteen hours before the
person who needed them read them.

**7. For coordinate-bound lock types — `spot-difference`, `pin-drop-map`,
`readout-map` — art must exist before the answer is authored.** `spot-difference`
is the live proof: its coordinates were keyed before any image existed, and
nobody can now confirm they land on a real difference without reading the
answer key. **Build the authoring tool where the answer is derived by clicking
real art**, rather than a process rule; Lock Library's point is that a process
rule survives right up until someone is in a hurry.

**8. An unreferenced-asset check.** `doctor` reports images referenced and
missing; nothing reports images present and referenced by nobody, which is why
twelve accumulated invisibly.

**9. Confirm the browser works before the first commit.** An evening of
verification was borrowed from another session for want of a two-minute check.

---

## What this review cannot tell you

Website Designer's account had not arrived. Website Infrastructure never worked
the Tailwind branch and says so — their contribution is corroboration and
prediction, not history. Operation Builder's own early hours are reconstruction
from git because their context was compacted mid-build, and they flag which
parts are inference. **The hotspot rebuild was done by a session that is no
longer running and has left no retrospective.**

**And the largest untested assumption is not in this document.** Nobody has
ever played a Bureau Operation with real children. Every judgement here is
about how the thing was built, not whether it works in a classroom.
