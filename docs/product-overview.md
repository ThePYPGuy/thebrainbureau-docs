# Brain Bureau — Product Overview

**Status: locked.** This is the canonical language. Where code and this
document disagree, this document is right and the code is behind.

Last agreed: 2026-08-24.

---

## The four activity types

### The Case — multi-lesson narrative investigation

The flagship type. Students are agents working a real-world scenario across
several lessons, using subject content as the actual investigative tool.
Phases unlock automatically as students progress, with a teacher-controlled
ceiling so nobody outpaces the curriculum.

Contains auto-checked tasks, an **Evidence Board**, and one written **Final
Brief** with self-assessment against an exemplar. Documents — suspect
profiles, intercepted notes, invoices — unlock as tasks complete and print as
a physical dossier.

Example: **Operation Cipher**, a financial fraud investigation using Y6 Four
Operations.

### The Operation — single-lesson escape room

A self-contained puzzle sequence finished in one lesson. **Sequential tasks,
each gating the next**, building to a final code or reveal. Fully
auto-checked. No documents, no phases, no teacher involvement during play.
The fastest type to build and the best entry point for a new teacher.

**The defining constraint is one lesson, not a task count.** Four tasks or
seven are both fine; what makes something an Operation is that a class starts
and finishes it in a single sitting. Around five is the natural size for a
45-minute lesson, and a useful default when authoring, but it is guidance
rather than a rule.

**Call them locks.** An Operation is an escape room and the site says so, so
the unit a student works through is a **lock**, not a phase or a task. That is
the word in the interface, on the teacher dashboard and in marketing.
Internally locks are still phase rows — see the note on gating below — but
that plumbing never surfaces in language a teacher or child reads.

**Each Operation is a standalone.** Its own narrative, its own visual
identity, its own artwork. They are not variations on a template, and the
platform's job is to frame them consistently rather than make them look alike.
See [visual-identity.md](visual-identity.md).

Example: **The Vault** — four locks, each releasing one digit of a final
code, covering Y6 Four Operations.

### Agent Training — live, whole-class quiz game

The **only synchronous type**. The teacher launches a live session, students
join on their devices, and the class plays together in real time.

Teachers build **question banks** — AI-assisted from a topic or pasted text,
teacher-reviewed before use — which run across multiple sessions and classes.
Banks are **mode-agnostic**: a bank built for one mode runs in any future mode
without rebuilding the questions.

First mode: **Mainframe Breach**. Answer correctly to earn **access tokens**;
spend tokens on attacks, defences and intelligence moves against other agents.

### The Field Op — real-world scavenger hunt

Students move through physical spaces — the school building, a museum, an
outdoor site — completing location-based or object-based tasks. Closest in
spirit to Goosechase. Cross-curricular and experiential. Phase 2.

---

## How they fit together

| Type | Format | Live or async | Duration | Phase |
|---|---|---|---|---|
| The Case | Narrative investigation | Async, self-paced | Multiple lessons | 1 |
| The Operation | Escape room | Async, self-paced | One lesson | 1 |
| Agent Training | Quiz game modes | Live, whole-class | 15–30 min | 2 |
| The Field Op | Scavenger hunt | Async, real-world | One session | 2 |

---

## What runs across all four

**Intel** — platform-wide cumulative currency earned on every activity.
First-attempt-correct earns more than later attempts; hints carry a small
penalty; completion and clean-sheet bonuses on top. Feeds agent clearance
levels. See [intel-and-clearance.md](intel-and-clearance.md).

**Agent profiles** — persistent pseudonymous accounts. Codename plus PIN, no
email, no real name. The teacher keeps the roster mapping; the platform never
holds identifying data by default. Student SSO is a school-level opt-in. See
[identity-and-access.md](identity-and-access.md).

**Teacher dashboard** — four clearly labelled panels, one per activity type
(Operations, Cases, Agent Training, Field Ops), plus a live view showing the
status of whatever is currently running. Who has joined, which task each agent
is on, attempts per task, open tasks awaiting conference. One screen, no
clutter.

**Curriculum tags** — every task tagged with a plain-language skill
description plus framework references across UK National Curriculum, Common
Core, Cambridge Primary, IB PYP and Australian Curriculum. Teachers filter the
activity library by framework, year group, subject, learning goal and title.

**Entities** — School → Teacher → Class → Deployment → Agent. One purchase
gives a teacher permanent entitlement to an activity; they deploy it as often
as they like to different classes.

> **Note:** the original four-entity model omitted Class. It was added on
> 2026-08-24 because assignment, the student dashboard and live-game rosters
> all need real class membership rather than a shared text label.

---

## Where the current build diverges

Recorded honestly so the gap is visible rather than discovered later.

| Divergence | Detail |
|---|---|
| ~~Global Intel Cards is mislabelled~~ | **Fixed 2026-08-24.** Now `case`, subtitled "Case 01"; Zero Hour became "Operation 01" as the only Operation. |
| ~~Locks are called phases in the UI~~ | **Fixed 2026-08-24** in the student terminal: an Operation shows LOCKS and "LOCK 01 OF 5", a Case keeps its Mission Map. The word "phase" still reaches the *teacher* dashboard. |
| **Agent Training is async** | What shipped is a solo game against simulated nodes. The real type is live and whole-class. Reconciliation: the shipped game becomes one *mode*, and its quizzes become *question banks*. See [question-banks.md](question-banks.md). |
| ~~Intel is one number~~ | **Fixed 2026-08-24.** `intel_earned` drives clearance and only rises; `intel_balance` is what the shop will spend. |
| ~~Hints are free~~ | **Fixed 2026-08-24.** A hint costs 25 against that lock's award, `hints_used` is written, and the text is fetched rather than shipped so the charge cannot be dodged. Bonuses are untouched. |
| ~~Clearance never changes~~ | **Fixed 2026-08-24.** Six ranks of three stars, promotion announced where it happens. |
| **Curriculum tags are per-activity** | One framework, no per-task tags, no library or search. |
| ~~Everything is owned by everyone~~ | **Fixed 2026-08-24.** Three tiers — subscriber, purchaser, unentitled — in `lib/entitlements.ts`. The library shows the whole catalogue with unowned titles locked, and deploying requires an entitlement at the database level. |
