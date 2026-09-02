# Lock Library spec — review

**The architecture is right and I would build it.** Room choice + a lock-instance
file + copy + props really is the one-day Operation, and the two halves fit: the
Room Variations Library defines the furniture grammar, the Lock spec binds to it.

Everything below is a problem I can point at evidence for, gathered from the
Tailwind build on 2 September rather than from principle.

---

## 1. Two "Built" claims are false, and that is the expensive kind of wrong

**`drag-sequence` is NOT built.** The spec says *Built (Encore first draft)*.
`grep -rln Encore lib components app content` returns nothing. There is no
Encore in this codebase.

**`table-lookup` is NOT a component.** The spec says *Built (Prime Directive)*.
There is no data-table component; it is inline JSX in `Tasks.tsx` with one real
call site.

**This is the third time this exact error has appeared in a planning document.**
The Tailwind handoff claimed four reusable components and one was real
(`Facsimile.tsx`). The series bible's §4.1 had three of seven "already built"
rows wrong. A day of the Tailwind build went on discovering it.

What is actually there is better than what the spec claims, which makes the
error more annoying rather than less: **`OrderingTask` exists** and is the right
starting point for `drag-sequence`'s DATA. **CORRECTED 2 Sep: I also wrote that
it is already tap-first, which the drag ruling requires. It is not** —
`Tasks.tsx:512` renders numeric rank boxes, so there is no drag and no tap-to-
reorder to promote. **The interaction is new work.**

**That is a FOURTH planning document wrong about this codebase, and this one is
mine** — written into §1, the section about exactly this error, and copied into
the plan, the brief and Lock Library's inbox before anyone checked it. I
relayed it from a build report rather than running the grep the section
demands. Found by the `drag-sequence` agent.

**Recommendation:** §7.1's audit should carry an explicit default — **a claim
that something is already built is treated as false until a grep names the file
and a browser draws it.** Not scepticism for its own sake; three for three.

---

## 2. THE BIG ONE: nothing in the contract requires a validator to ACCEPT

Contract item 2 is *validation*. Every type's line says how it refuses. **No
line anywhere says a type must be shown accepting a correct answer.**

Three things went wrong this way in one build:

- The console gate has only ever been exercised **at zero findings**, where
  refusing is the correct behaviour. OB's own words: *refusing when it should
  refuse is half a test.*
- A 403 entitlement probe **passed while testing nothing** — the request had
  failed to sign in, and a failed sign-in answers 401, which reads like a refusal.
- `matching` was **unit-proven, guard-proven and never drawn.** Three green
  checks, none of which would notice that it does not render.

A library of twenty types can ship entirely green with several of them refusing
everything, and the gallery will not catch it, because **the gallery demo is
authored by the same person who wrote the type, with a config chosen to work.**

**Recommendation:** make contract item 2 read *validation, with at least one
case that must be ACCEPTED and one that must be REFUSED, and a browser render
that has been driven by a real click.* Add "drawn in a browser" as a seventh
contract part, because unit tests demonstrably do not imply it.

---

## 3. The answer-space guard shares its constant with the thing it guards

Type 20 *computes and enforces the answer-space product at author time*.

**A second gate derived from the first gate's source is one gate.** If the space
is computed by multiplying declared cardinalities from the same config that
defines the parts, then any authoring mistake that shrinks the REAL space is
invisible to it. Type 7's own `locked_slots` is the worked example: lock two
slots and 5! is no longer 120, but a product of declared cardinalities still
says 120.

**The fix is already in Tailwind's acceptance checklist and the spec did not
inherit it.** §9 asks that the console reject all **479** wrong combinations —
which is 480 minus the one that works. **Enumerate the accepted set.** At these
sizes it is trivial, and it is a stronger check than a multiplication because it
tests the validator rather than the config: exactly one member accepted, every
other refused, identically.

That single test satisfies §2 above and §3 at once, for every finale.

---

## 4. Persistence is not in the contract

Contract item 6 says every solved instance posts a finding. Good. Nothing says
the instance's own state survives a reload.

Migration 37 created `agent_findings` AND `agent_board_state`, each with an
index and RLS, and **neither had a writer.** A lock that renders and grades
looks perfect until a child logs out.

**CORRECTED 2 Sep, and the correction is the more interesting half.** Both were
fixed the same day: `tailwind:lib/server/board-state.ts` is a real reader and
writer over a tested pure whitelist in `lib/engine/board.ts`. The sentence
above was true of `main` and false of the project.

**This document told you to distrust exactly this kind of claim, and then made
one.** Lock Library caught it by applying §1's own rule — *a claim that
something is built is false until a grep names the file* — to the document that
taught it the rule. **Contract item 8 stands; its evidence has been overtaken.**

**Recommendation:** contract item 7 — *persistence proven by a round trip:
solve, reload, still solved.* A table nobody writes to is not a feature, and it
fails by looking exactly like one.

---

## 5. Config-driven authoring hits the import problem head-on

§5 is the point of the whole spec, and it is the right idea. It also walks
straight into the defect that has bitten this repo twice.

Adding a column to `activities` is **four** coordinated edits, and the fourth —
the `on conflict do update` list — is silent: the value populates on first
import and then never updates, while `content_hash` still moves, so
`deploy:check` reports the database matches the repo.

A join table is worse, because there is no conflict clause to forget:
**re-import ADDS and never DELETES.** Remove a lock instance from the file,
re-import, and the row stays.

**Recommendation:** state in §5 that importing an Operation's instances is a
**replace, not a merge** — delete the Operation's rows and insert the current
set, in the same transaction. And prove it by **removing** an instance and
re-importing, never by adding one.

---

## 6. Projector mode doubles twenty surfaces and would be tested on none

Contract item 3 requires both delivery modes for every type. Class mode is
deferred for Tailwind, so today nothing exercises the projector path.

Building it for twenty types and testing it for zero produces twenty half-done
things that read as done — which is precisely the trap already sitting in
Tailwind's checklist, where *board state survives logout/login (both modes)*
does not read as a class-mode line and would be ticked by anyone testing solo.

**DECIDED 2 Sep: teacher-panel entry is OUT of wave one.** So contract item 3
drops to *solo delivery only* for the twenty, and the projector path becomes a
second wave with its own contract item and its own proof.

**Write that down in the spec rather than leaving it understood.** A type whose
config carries a `teacher entry` line that nothing implements is a claim, and
this document's §1 is about what claims cost. Either strike the teacher-entry
line from all twenty type definitions, or keep it and mark the whole column
**wave two** at the top of §2 so no one reads it as built.

The reason this matters more than it sounds: Tailwind's own acceptance list has
*board state survives logout/login (both modes)* sitting in it, and the half
that defers is invisible in the wording. **A deferred half hides inside a line
that does not mention it.**

---

## 7. Smaller things

**`store` and `flavour` have no lock binding.** The room grammar names five
regions; §3 consumes three. Yet every instance carries a `reference_file`
pointer and the store is *the Reference Files home*. So there is a pointer with
no rendering contract. Either bind it or say plainly that Reference Files are
placed by the room and never by a lock.

**Series-wide behaviours need an enforcer, not a convention.** *No lockouts, no
partial feedback, one hint* are listed as inherited. Twenty independently
written types will diverge unless the engine wraps every lock and owns those
three. Make them the wrapper's job.

**`npm run locks` should flag ZERO usage, not just count.** `npm run skins`
taught us the distinction that matters: `situation-room` was **legal in the
CHECK constraint and unworn**. A type nobody uses is a type nobody has tested in
anger, and it will be discovered by the first Operation that needs it.

---

## 8. The archetype backgrounds

Sixteen renders plus the prompt library, and the grammar is consistent — every
scene maps canvas / rail / store / station / flavour, which is what makes the
Lock spec's §3 possible at all.

**Two copies of a room is two rooms.** The Green Archive ships portrait and
square; the Red Archive landscape and portrait. If regions are measured per
file, those pairs will drift, and drift silently — a region twenty pixels out
looks like a slightly untidy composite, not an error. This is the same problem
the inset maps had, and the answer there was good: *not copies, re-derivations,
from one source, with integrity asserts.*

**Only one room is measured.** The prompt library says SR-1 is *already rendered
and measured* — the Tailwind chassis. Fifteen are not, and measuring is the
manual step that will become the bottleneck the library was meant to remove.

**Recommendation:** every room's `scene-layout.json` gets a visual-regression
baseline. The harness already exists (`tests/visual/baseline/`). A wrong region
is a visual fault and only a visual check will catch it.

**And these are 3-4 MB PNGs, sixteen of them.** A four-megabyte background per
Operation is a real cost on a school connection. They are painterly renders, so
SVG is not the answer — AVIF or WebP with responsive sizes is. Worth settling
before fifty exist rather than after.

---

## 9. On "built up front, in full"

Twenty types times six contract parts, plus twenty gallery demos in two modes,
is a large sprint. Tailwind is one Operation using about five of these types,
with a dedicated session on it all day, and it is not finished.

**I am not arguing against the investment — I am arguing about the order.**
Build the types Tailwind has already proven in a browser first, by promoting
working code: `readout-map` and `pin-drop-map` (both proven today, real clicks,
answers stripped from the client), `matching-pairs`, `verdict-select`,
`composite-finale`. Those five are the ones where §7.1's audit will find
something real. The other fifteen are new code wearing a familiar name, and
should be scheduled as such.
