# Roadmap

**Agreed 2026-08-24.** Ordered by dependency, not by appetite — several of the
priorities below cannot be built well until the foundation beneath them
exists.

Your stated priorities are marked ★.

---

## Stage 0 — Foundations

Everything else hangs off these, and each gets more expensive with every week
of real classroom data.

**Agents are school-scoped.** ✅ Built 2026-08-24. A codename now belongs to a
school, so a child taught by two teachers in one building is one agent with
one Intel total. Two partial unique indexes rather than one, because a teacher
without a school is still a valid state: where a school is set it owns the
namespace, and where it is not, behaviour is unchanged. Verified by joining
the same codename through two teachers in one school and getting one agent
back.

**Class is a real entity.** ✅ Built 2026-08-24. A class has a name, a
persistent join code and real members; deployments attach to one. Entering any
code in a class makes you a member of it permanently, which is what puts a
child on the register before they have started anything. The migration
backfills classes from the old `class_name` labels — replayed against real
data, every deployment landed in a class of the same name, nothing orphaned.

> One limitation of that backfill: membership can only be recovered for agents
> who left progress behind, since progress is the only evidence a historical
> join happened. Anyone who joined but never answered anything re-joins the
> first time they use a code again.

✅ The student-facing side is done too: joining by class code, leaving a class
from the dashboard, and every door landing on the assignments page.

✅ **Intel splits into two numbers.** `intel_earned` drives clearance and only
ever rises; `intel_balance` is what the shop will spend. Done before anyone had
spent anything, which was the whole point -- sharing one column would have meant
buying an avatar demoted you.

✅ **Entitlement tiers replace the blanket grant.** Built 2026-08-24.
`@eishcmc.com` is a subscriber and holds the published catalogue,
`maciejborucki@googlemail.com` is an individual purchaser holding Zero Hour
only, everyone else is unentitled. The policy is three constants in
[lib/entitlements.ts](../lib/entitlements.ts) — deliberately, so that deleting
them is the whole job when a payment webhook replaces it.

Grants are **additive**. A sign-in never revokes, so a real purchase survives
any later edit to the dev policy and a teacher mid-term cannot lose access to
something they are actively teaching because a constant changed. Revoking is a
migration's job, where it can be reviewed.

The library lists the whole catalogue with unowned titles locked rather than
hidden, since something invisible can never be wanted. The purchaser account
deliberately does **not** own everything — an account that owns the catalogue
can never see the locked state it exists to exercise.

Locked now means locked in the database, not only in the interface: deploying
an activity requires an entitlement in RLS, not just in the server action.

A subscriber carries a **FULL ACCESS GRANTED** stamp in the dashboard topbar,
a purchaser **LICENSED ACCESS**, and an unentitled account none — so the kind
of access an account holds is readable on arrival rather than inferred by
counting the library.

#### Ready for real purchases and subscriptions

Checked deliberately, because rebuilding this after money has changed hands is
the expensive version:

- **`entitlements` is the source of truth and is enforced in RLS**, so a
  payment webhook writing rows is the whole integration. Nothing else has to
  learn about payments.
- **`unique (teacher_id, activity_id)`** makes a webhook retry idempotent
  without any extra bookkeeping.
- **`source` separates a subscription from a purchase.** This is what makes a
  lapse survivable: cancelling deletes `source = 'subscription'` rows and
  leaves purchases standing. There is a test for exactly this — buy a title,
  add a subscription over it, lapse the subscription, and the bought title is
  still there. Without that separation a cancellation would delete things
  people had paid for.
- **Grants are additive and top up on sign-in**, so publishing a new activity
  reaches every existing subscriber without a backfill.
- **The policy is one module.** Replacing `lib/entitlements.ts` with a real
  lookup touches nothing else.

#### Not ready, and what each needs

- **There is no subscription record.** The tier is derived from an email
  domain, which cannot express *expiry*, *plan*, or *cancelled on 31 August*.
  A real one needs a row with a status and a period, plus something that acts
  when it ends.
- **Subscriptions should belong to the school, not the domain.** Domain
  matching happens to cover a single-domain school and breaks for a teacher
  using a personal address, a school with two domains, or a trust with many.
  `schools` is the right home.
- **There is no order record.** No transaction id, no receipt, no amount —
  so nothing can answer "did this person pay, and for what" beyond the fact
  that a row exists.
- **Agent Training is outside all of it.** `training_sims` is a separate
  table with no entitlement relationship, so question banks cannot currently
  be sold or restricted. If banks are ever a product, that is a second
  entitlement system unless they are brought under this one first.
- **Nothing revokes.** A lapse is a `delete ... where source = 'subscription'`
  that no code performs yet.

---

## Stage 1 — Access and dashboards

★ **Teacher signup, smooth end to end.** Currently works but has never been
walked as a first-time experience: school selection, entitlements, first
class, first deployment, first code to hand out.

★ **Three student access types.** ✅ Front door, class code and deployment code
all built — see [identity-and-access.md](identity-and-access.md). Guest,
one-session play is still to do.

> ✅ **PIN reset built 2026-08-24**, with a change-your-own-PIN companion so a
> child is not stuck with a random one. Still outstanding: display-name
> moderation, which the guest tier will need, and ending live sessions on
> reset — see [identity-and-access.md](identity-and-access.md).

**Student dashboard.** ✅ Built at `/profile`, in the light face, with the
avatar and store slots standing empty rather than mocked up.

✅ **Teacher dashboard, four panels.** Built 2026-08-24. Operations, Cases,
Agent Training and Field Ops, with **Running now** above them — open
deployments and their join codes, first on the page because it is the only
part that is time-sensitive: a teacher standing in front of a class needs the
code before anything else.

The library and the panels are one structure rather than two. An earlier cut
had a type-grouped library sitting under a set of panels, which listed every
activity twice and left neither obviously authoritative. Locked titles sit in
the panel for their own type, where "another Operation" is a more useful
suggestion than "another thing".

Field Ops shows honestly that nothing exists yet rather than an empty frame
implying something is missing.

✅ **Clearance levels.** Six ranks of three stars -- eighteen rungs -- in
[lib/clearance.ts](../lib/clearance.ts),
thresholds as plain configuration rather than a table, so tuning a guess does
not need a migration. Clearance is *stored* on the agent, not computed, so
lowering a threshold later can never demote a child who was already promoted
under the old one. Promotion is a visible event: `awardIntel()` is the single
place Intel is granted, it returns the rung crossed, and the mission and
training debriefs show a banner. The dashboard carries a progress bar to the
next rung.

✅ **Hint penalties.** A hint costs 25 against that lock's award, taken off
the base only -- the completion and clean-sheet bonuses are for finishing and
for accuracy, and asking for help is neither a failure to finish nor a wrong
answer. Priced deliberately below a wrong guess (which costs 75), because a
penalty steep enough to deter asking teaches a stuck child to guess instead.

The text is now fetched from `/api/hint` rather than shipped with the task:
the same call reveals it and records the charge, so a cost avoidable by
reading the page source is not left avoidable. Charging is idempotent, and the
hint survives a reload -- billing a child twice for something they already
paid for would be indefensible to explain.

✅ **A deploy check that makes silent drift visible.** Built 2026-08-25.
`npm run deploy:check -- --prod` compares a database against the repo:
migrations not applied, content files edited but never imported, an empty
crosswalk, untagged tasks.

It exists because the same failure had already shipped twice. Publishing has
three parts and only one is automatic, and the two manual ones -- migrations
and content imports -- fail without erroring: the app reads the old row and
carries on. A hint penalty that charged nothing and curriculum tags that
nothing could search both looked deployed and were inert.

Content drift is detected by a canonical hash the importer stamps on the row,
so key order, indentation, line endings and `_`-prefixed notes are not
mistaken for changes. Real drift exits non-zero; things it could not confirm
are reported separately, because a check that cries wolf gets ignored.

---

## Stage 2 — Content and the library

★ **Question bank structure.** Split bank from mode from session, per
[question-banks.md](question-banks.md). Do this *before* many banks exist.

🔶 **Curriculum tagging and search.** Tagging built 2026-08-25; search is not.
The crosswalk imports into tables, tasks carry skill ids, activities carry an
age band, and every label resolves into the reading teacher's framework —
"Year 6" in London, "Grade 5–6" in Boston, from the same tag. See
[curriculum-tagging.md](curriculum-tagging.md) for the two gaps found in the
crosswalk while tagging real content, and for why the promised "Extension"
alignment state was not built.

✅ **Skin architecture.** Built 2026-08-24, while it was still cheap. An
activity's `skin` now reaches the DOM: `loadStudentState` carries it, `Frame`
puts it on `data-skin`, and every colour in `globals.css` resolves from a
token defined against that attribute. Changing one database column changes
the rendering with no code touched — verified by switching Zero Hour to
`case-file` and back.

Two layers of token, deliberately. A semantic set (`--accent`, `--ink`,
`--surface`…) is the contract a new archetype implements; the original
colour-named tokens (`--cyan`, `--amber`…) are now aliases onto it. They stay
because ~120 rules already use them, and renaming all of those in the same
change would have meant one commit that both restructures and repaints, with
nothing to bisect if the result looked wrong.

Chrome is the other half, chosen by the same value. Only Field Terminal has
real chrome — bezel, scanlines, boot sequence. The other six get a plain
themed surface: wrapping a paper dossier in a monitor bezel would look
intentional, which is worse than looking plain.

[lib/skins.ts](../lib/skins.ts) is now the single archetype list — the
`npm run skins` script reads it rather than keeping its own copy — and it
records which have actually been *designed*. One has. `case-file` carries a
provisional palette, enough to prove a second archetype really is a token
block and no more; it still needs drawing. See
[visual-identity.md](visual-identity.md).

★ **Five Operations.** Authored as JSON content files, each standalone with
its own narrative and visual identity. Operations are defined by fitting one
lesson, not by a task count. Their units are **locks**. Do the skin work
first, or each one accumulates one-off CSS.

★ **Operation Cipher.** The flagship Case, and the first real test of Case
machinery: Evidence Board, unlocking documents, printable dossier, and the
Final Brief with self-assessment against an exemplar. None of that exists —
this is a build, not just content.

~~**Relabel Global Intel Cards as a Case.**~~ ✅ Done 2026-08-24, forced by the
locks wording: leaving it typed as an Operation made a six-lesson Case tell
students it had LOCKS. Zero Hour is now "Operation 01" as the only one.

---

## Stage 3 — Agent Training, live

★ **Signal Check, then Mainframe Breach** (ordered 2026-08-31). Both are modes
under Agent Training; neither replaces the other.

Signal Check is first and deliberately thin — one question to the whole class,
a fixed answer window, no leaderboard. Its job is the machinery: session
lifecycle, game PIN, realtime transport, host controls, reconnection, late
joiners. The async modes forgive dropped wifi and dead iPads; this does not,
which is the bulk of the work, and doing it inside the thinnest possible game
keeps a broken transport from looking like a broken game.

Mainframe Breach follows, adding its token economy on top of a transport that
has already carried a real class. This entry named it alone while it was the
live mode; `question-banks.md` holds both modes in full.

**AI-assisted bank generation**, teacher-reviewed before any child sees a
question.

---

## Later

Field Ops. The shop and what Intel actually buys. Student SSO as a school
opt-in. Payment and real entitlement granting. Year-end rollover — classes
archive, agents persist, teachers start again. Accessibility mode: high
contrast and a plain font, because green-on-black with a pixel typeface and
red/green feedback is a hard read for dyslexic and colour-blind pupils, and
schools ask during procurement.

---

## Settled

- **School is required at teacher signup** (2026-08-24).
- **Class membership is a real entity** (2026-08-24).
- **Operations are defined by lasting one lesson**, not by a fixed task count
  (2026-08-24).
- **Crosswalk skills have stable, append-only ids** (2026-08-24).
- **The Bureau face is light; the Field stays dark** (2026-08-24). Built. The
  logo keeps its dark plate.
- **Seven skin archetypes, locked** (2026-08-24). A constrained `skin` column
  plus `npm run skins` to catch repetition before it ships.
- **An Operation's units are called locks** (2026-08-24). Phase rows
  underneath; the word never surfaces.
- **Task-level gating is not a priority** (2026-08-24). Phases gate well
  enough, each Operation is designed standalone, and Global Intel Cards works
  as it is. Revisit only if a specific activity needs strict order inside a
  phase.

## Open questions

1. Should a Type 3 agent be barred from the front door, or is it one record
   with two doors? Recommended the latter.
2. ~~Do Agent Training **access tokens** convert to Intel?~~ **Decided
   2026-08-31: they convert.** Whether *earned* or *unspent* tokens convert is
   open; see `intel-and-clearance.md`.
3. Clearance thresholds and the three titles above Senior Analyst — proposals.
4. Whether the crosswalk grows a Humanities section, or non-covered subjects
   carry plain-language tags with no framework mapping.
