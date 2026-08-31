# Identity and Access

**Status: decided 2026-08-24**, except where marked OPEN.

Covers who can be on the platform, how they get in, and what the platform is
willing to know about them.

---

## The rule that shapes everything

The platform holds **a codename and a hashed PIN, and nothing identifying**.
Teachers keep their own roster mapping codename to child, stored separately
from the agent record. This keeps COPPA, the UK Age Appropriate Design Code
and school data-processing agreements out of scope by construction rather than
by policy.

Student SSO (Google / Microsoft 365) is planned as an **additional** route,
opt-in per school — never the default, because school tenants block
third-party sign-in until an IT administrator approves the app, and a teacher
who buys on a Sunday must be able to teach on Monday.

Email and password are **not** a student sign-in route. Where they were
floated, the underlying need was PIN recovery, which is solved by teacher
reset instead.

---

## Telemetry — what leaves the building

**Decided 2026-08-31.** Error monitoring is needed before the first classroom
session: with thirty children playing, the current detection method is a child
raising their hand. It reports to a third party, and **identifiers are stripped
at the client, before transmission**.

**Codenames are only non-identifying outside the classroom.** Inside it everyone
knows who AGENT FALCON is — that is what the teacher's roster is *for*. So
"a codename is anonymous" is true of a vendor's servers and false of everyone in
the room, and the question a school asks at procurement is not *is this
technically anonymous* but *what leaves the building*.

Guest nicknames settle it regardless. They are free text typed by children, are
sometimes the child's own name, and no policy about codenames covers them.

**Sent:** the error and its stack trace, browser and device, question type and
question id, the session's mode and phase, and the sequence of **event types** —
`question_served`, `answer_submitted`, `reconnect` — carrying no payloads. That
is enough to debug a live session and identifies nobody.

**Stripped before it is sent:** codenames, guest nicknames, agent ids, game
PINs, session ids, and the answer text a child typed.

**A pattern cannot match an arbitrary word.** Codenames, session ids and PINs
have shapes. A child's typed answer does not — `photosinthesis` is
indistinguishable from any other string, and no rule can find it. So a surface
that collects free text must **register its identifiers as early as it has
them**, and a guard must assert that it did. Left to be remembered, this fails
the way everything here fails: silently, with the redaction visibly working on
everything it was told about.

There is a worked example, kept rather than tidied away. A dev-only probe page
never called the registration, and its Sentry title reads *no run for
[redacted] in session [redacted] (pin [redacted]) answered "photosinthesis"* —
codename, session and PIN all removed because each has a shape, and the typed
word untouched because it has none. A fixture rather than a child, on a
`development` event. It is exactly the leak this rule exists to prevent, and the
structural patterns did everything they could.

**Two things reach Sentry that the client cannot strip, and the difference
between them decides the fix.** Found 2026-08-31 on the deploy-verification
event, after the contract below had been written.

`user.geo` — city level, *VN, Tan An, Vietnam* — is derived by Sentry from the
connecting IP **at ingest**, after everything this codebase controls. No
`beforeSend` can reach it. It goes off in the project's settings or not at all.

`contexts.culture` — locale, calendar and **timezone** — is collected by the
browser SDK *before* the send, so it is within reach and can be dropped in
`beforeSend` like anything else.

Both matter for one reason: on a platform whose model is a codename and nothing
identifying, the town a child's device connected from and the timezone it sits
in are precisely what a school asks about at procurement. And they fall outside
the frame this section had drawn — **"stripped at the client" is a complete
answer only for what the client sends.** The rule was right and its scope was
not.

**At the client, never in the vendor's filters.** Server-side scrubbing means
the data arrives and is then deleted, which is a promise about somebody else's
behaviour rather than a property of this system. Stripping at the client means
it never leaves the device. That is the same distinction as the rule at the top
of this file — by construction rather than by policy — and it is the half a
school can be shown rather than told.

**This needs a guard, because it is a silent leak.** A check failing when a
payload carries a codename pattern or a PIN: a scrubber that regresses sends
real identifiers and looks exactly like one that works. It belongs beside the
answer-key leak check, which is the same shape.

**The guard must spell out its own patterns rather than importing the
scrubber's.** A test derived from the thing it tests is one gate, not two:
break the scrubber and the guard's list empties with it, and it passes having
checked nothing. `CLAUDE.md` records this happening twice already.

**Settled at setup, 2026-08-31.** Sentry, **EU region** — so that *it does not
leave Europe* is the whole answer to a school asking where the data lives,
rather than an explanation about scrubbing. The region is fixed when the
organisation is created.

- **Browser requests tunnel through the app's own domain.** School networks
  filter telemetry hosts aggressively, and a blocked report is a silent absence
  — monitoring that works everywhere except the classroom it was installed for.
  The same reasoning that put the fonts on `next/font/google`.
- **Session Replay is off, and that is a decision.** It reconstructs what was on
  a child's screen, and **the scrubber does not reach it**: the guard would go on
  passing while the thing it guards against arrived by another channel. It is
  defensible later on teacher-facing screens only, and never on `/live/play`.
- **Tracing is on**, which puts route URLs and navigation breadcrumbs into the
  payload. Signal Check's routes carry a **session id**, which is on the strip
  list above. So the scrubber has to cover transactions and breadcrumbs and not
  only errors — and be proven against each by breaking it, not by inspection.
- The build-time auth token lives in **Vercel's environment**, never in the repo.
  Without it a production build still succeeds and silently skips source maps,
  so every stack trace arrives minified: installed, reporting, and useless.

  **The build log says so before any error exists.** Source maps upload during
  the build, so a wrong or missing token shows up in Vercel's log minutes after
  a deploy — no need to wait for something to break and then read a stack trace
  to find out. Read the log first; confirm with a real error second. The two
  answer different questions, and only the first one is available immediately.

Self-hosting is the eventual answer if schools press on it. It is a running
service to maintain, and it is not needed yet.

---

## Entities

```
School
  └── Teacher
        └── Class ──────── Class Membership ──── Agent
              └── Deployment (one activity assigned to one class)
```

- **School** — owns the subscription. Agents are scoped here.
- **Teacher** — belongs to a school. Owns classes and deployments.
- **Class** — a real entity, not a text label. Has a name, a persistent join
  code, and members.
- **Deployment** — one activity assigned to one class. Has its own code.
- **Agent** — a pseudonymous student account, unique per **school**.

> **Built 2026-08-24.** All three doors work: deployment code, class code and
> the front door. Agents carry `school_id`; classes have members; students
> join and leave classes from their own dashboard.

### Why agents are school-scoped, not teacher-scoped

Today `agents` is unique per `(teacher_id, codename)`. A child taught by two
teachers in the same school gets **two agents and two separate Intel totals**.
Since Intel is explicitly platform-wide and cumulative, that is a defect, and
it gets more expensive to fix with every week of real data.

Scoping to `(school_id, codename)` means one child, one agent, one Intel
total, across every teacher and class in their school.

**Decided 2026-08-24: a school is required at teacher signup.** School
selection already exists in the signup flow as an optional search-or-create
field; it becomes mandatory. This is what makes school-scoped agents possible
and what the subscription model needs anyway, and it costs a teacher nothing
at the one moment they expect to be filling in details.

---

## The three student types

Three doors, two storage tiers. Types 1 and 3 are **the same record**; what
differs is whether the student ever uses the front door.

### Type 1 — Agent with an account

Signs in at the front door: **school + codename + PIN**, or school SSO where a
school has opted in. Lands on the student dashboard. Can be a member of
several classes, sees everything assigned to them, keeps Intel, clearance,
avatar and results.

### Type 2 — Guest, one session only

No account. Enters a deployment or game code, types a **display name**, plays,
and the result is recorded against that name for the teacher to see. Nothing
persists; no PIN; no Intel; no avatar.

Permitted for **Operations** and **Agent Training**. Refused for **Cases**,
which span lessons and would lose everything between them. The activity
declares whether it fits one sitting; the join screen refuses and explains
rather than letting a child lose a term of work.

### Type 3 — Agent via class code, never uses the front door

Enters via a class or deployment code with codename and PIN. Progress,
Intel and clearance all persist exactly as Type 1. This is Type 1's record
reached through a different door — the child simply never signs in at the
front.

**OPEN:** whether a Type 3 record should be flagged so it *cannot* use the
front door. Recommend not — one record, two doors, no flag.

---

## Teacher handles — decided 2026-08-31

**A teacher publishes under a handle, never under their name.** Compulsory at
signup, unique across the platform, and the only thing another teacher sees on a
public bank in *Discover*.

`teachers` already holds `email` and a nullable `display_name`, and
`display_name` arrives from the Google or Microsoft profile — so it is a real
name. That is fine where it is: the teacher's own dashboard, their own sidebar.
It is not fine on a searchable library every teacher on the platform can read.
**The handle is a second field, and the two are not interchangeable:**
`display_name` is what you are called; the handle is what you publish as.

**The signup screen has to say what it is for.** A field labelled *handle* with
no explanation gets a real name typed into it, which defeats the entire point
and cannot be taken back once a bank is public. Say it plainly at the moment of
choosing: *this appears on every bank you publish, and every teacher can see it.*

Two consequences that follow:

- **Unique platform-wide**, so collisions are common in a way codenames are not —
  those are scoped per teacher. Block at creation and suggest alternatives, the
  same shape as the codename rule.
- **Existing accounts have none.** Anyone who signed up before this needs to
  choose one, and the natural gate is the first attempt to publish a bank rather
  than a blocking prompt at next login.

Changing a handle later is allowed: it is rendered from the teacher row, so a
change follows everywhere it appears, and the uniqueness check is the same one.

---

## Codes

Three distinct things, deliberately named apart.

| Code | Purpose | Lifetime |
|---|---|---|
| **Class code** | Join a class and stay in it | Persistent, per class |
| **Deployment code** | Enter one assigned activity; the guest route | Persistent, per deployment |
| **Game PIN** | One live Agent Training session | Ephemeral, per session |

A teacher normally shares the **class code** once. Deployment codes cover
one-off use and guests; game PINs are Phase 2.

**The game PIN is six digits** (2026-08-31), and its uniqueness index covers
only *joinable* sessions rather than every session ever run. Four digits is ten
thousand, and a school running several classes in one period would start
colliding — a teacher reading out a PIN that drops a child into another class's
game is not a fault anyone diagnoses quickly.

**A signed-in agent is used only when the client asks for it** (2026-08-31). A
class set of iPads means the previous child is still in the cookie, so
defaulting to it would file one child's answers against another child's Intel —
and the child it happened to would have no way of seeing that it had. A live
join always states who is playing rather than inferring it.

---

## Registration, and why this shape

Kahoot is nickname-plus-PIN and almost entirely anonymous. Blooket adds an
optional student account with a username, password, persistent stats, avatars
and a shop. Gimkit leans on Google Classroom and Clever rostering — precisely
the IT-admin dependency ruled out above.

**Chosen: school-scoped accounts created by first use.** A student joins with
a class code, picks a codename and a PIN, and that *is* the account. No
separate sign-up, nothing to remember on day one, and no collision problem
because the namespace is one school rather than the world. Later they can sign
in at the front door with school + codename + PIN.

### Required supporting flows

- **PIN reset** from the teacher's roster. ✅ Built 2026-08-24. The teacher
  opens a child's page and issues a **generated** PIN, shown once and never
  again — only the hash is stored, so nobody can look it up afterwards,
  including us. Generated rather than teacher-chosen because a busy teacher
  sets the same PIN for the whole class, and a PIN thirty children share stops
  separating them at all. The child then changes it to something memorable
  from their own dashboard, which requires their current PIN: a shared device
  left signed in is the normal case, and the current PIN is what stops the
  next child at that desk locking the owner out.

  > **Not covered:** resetting a PIN does not end sessions already signed in
  > with the old one. Cookies last 12 hours, so an intruder with a live
  > session keeps it until it expires. Fine for "I have forgotten my PIN",
  > which is the real case; not sufficient if an account is ever actually
  > compromised. Needs a validity timestamp on the agent, checked server-side.
- **Guest display-name moderation.** Names appear on a shared screen during
  live games. Needs a profanity filter and a host kick, both of which Blooket
  and Kahoot have for the obvious reason.
- **Claiming.** A guest result, or an agent created in one class, should be
  attachable rather than stranded.

---

## Development subscription tiers

Real entitlement tiers, used while building so the UI for owning and *not*
owning can be tuned before anyone pays.

| Who | Treated as | Access |
|---|---|---|
| `@eishcmc.com` | European International School — full subscriber | Every published activity |
| `maciejborucki@googlemail.com` | Individual purchaser | Only specific purchased Cases and Operations |
| Anyone else | Trial / unentitled | Library browsable, activities locked |

This replaces the current behaviour, where every published activity is granted
to every teacher on first sign-in. That blanket grant must go, and the UI needs
a **locked state** and a **browsable library** — an unowned activity is
currently invisible rather than tempting.

---

## Student dashboard

**Built 2026-08-24** at `/profile`, in the Bureau's light institutional face
rather than the terminal's — this is where a child manages an account, not
where they do field work. The teacher's read-only preview of a child renders
the same component, so the two can never drift apart.

- **Agent identity** — codename, clearance, Intel, school. ✅
- **Avatar** — placeholder slot. ✅
- **My classes** — join by code, leave, see what each has assigned. ✅
- **Assigned work** — per class, with progress against each activity. ✅
- **Results** — mission and training history. ✅
- **Shop** — placeholder slot. ✅

Still to come behind those slots: the avatar system itself, and the store,
which spends the Intel *balance* rather than lifetime earned — see
[intel-and-clearance.md](intel-and-clearance.md). The dashboard shows lifetime
Intel today because the balance does not exist yet.
