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

## Codes

Three distinct things, deliberately named apart.

| Code | Purpose | Lifetime |
|---|---|---|
| **Class code** | Join a class and stay in it | Persistent, per class |
| **Deployment code** | Enter one assigned activity; the guest route | Persistent, per deployment |
| **Game PIN** | One live Agent Training session | Ephemeral, per session |

A teacher normally shares the **class code** once. Deployment codes cover
one-off use and guests; game PINs are Phase 2.

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
