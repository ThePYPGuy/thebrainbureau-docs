# Licensing and access

How a teacher gets to an activity, what a purchase buys, and what the free tier
is. **Decided 1 Sep 2026.** The commercial shape predates this file and was
spread across three places — `/pricing`, the `entitlements` table comment, and
`lib/entitlements.ts` — which is how it got forgotten once already.

---

## What is already true

**`/pricing` is live and promises this in public:** *free to sign up, one
purchase per activity kept for good, and a subscription for a whole school* —
and *an activity is bought once, not rented*. No figures are published yet, and
the page says so.

**The `entitlements` table has carried the shape since August:**

```
-- One purchase, unlimited deployments, permanent. Account-level, not per class.
create table entitlements (
  teacher_id  ...,
  activity_id ...,
  source      text not null,   -- 'teacherspayteachers', 'gift', 'trial'
  unique (teacher_id, activity_id)
);
```

`'teacherspayteachers'` was anticipated from the start.

**The read path works; the write path does not.** `lib/entitlements.ts` resolves
tiers, entitled slugs and lock reasons, and the dashboard already renders them.
Its own comment: *"This is a development policy, not a commercial one. Real
entitlements will come from a payment webhook writing `entitlements` rows."*
**Nothing writes an entitlement today.**

---

## The TPT code, and why it is deliberately shareable

**One code per activity**, shipped in the TPT download. Not per buyer — a static
PDF cannot carry a unique code, and any scheme needing the document updated is
ruled out.

**The code is a token; what it grants lives on the server.** The string in the
PDF never changes. What it unlocks is decided at redemption and can be re-tuned
whenever — tightened, capped, expired, moved to another tier — **without
republishing anything on TPT.** That is the whole reason this is safe to ship.

**Sharing is expected and is not a leak.** At zero users, a code that brings ten
teachers is worth more than the revenue lost from the nine who did not pay.
Somebody given the code gets a real, complete use of the product and becomes an
account. **Recorded as a decision so nobody builds anti-sharing machinery in six
months thinking they are closing a hole.** If one code is redeemed four hundred
times that is market research, and the response is a server-side dial, not a new
PDF.

*Available later, additively:* asking for a TPT order number at redemption and
checking it against the seller's buyer export — one order, one redemption. It
bolts onto the same flow and needs no change to any download. **Confirm what TPT
actually exposes to sellers before designing around it.**

---

## What redemption grants

| | |
|---|---|
| **The activity** | One class, up to 30 students, every year, **forever** |
| **Plus** | **One month of Full Access — everything on the site, no card** |
| **After the month** | Purchased activities + whatever the free tier holds |

**The permanent half keeps the published promise.** *Bought once, kept for good*
stays literally true; what is bounded is how many children it covers at a time,
which is how classroom resources are normally licensed and what a teacher
expects. **The cap is on students, not on uses** — metering deployments would be
renting, which `/pricing` says it is not.

**The month is the conversion mechanism**, and it lands after a teacher has
already run something with real children rather than before. `source = 'trial'`
already exists for it.

---

## The free tier must be data, not code

Its contents change without a deploy. Whatever holds it, `lib/entitlements.ts`
already treats the `entitlements` table as the single source and **fails
closed** — its own test names the danger: *"no entitlements read as
unrestricted"*. Keep that property.

---

## Admin

**A separate account with its own credentials** — decided 1 Sep. Stronger than
an `is_admin` flag on a teacher row, because with a flag the teacher's session
cookie *is* the admin credential, and any weakness in teacher auth would reach
the control that sets the free tier for every school. A separate identity makes
that failure impossible rather than mitigated.

- `/admin/*` checked server-side per request. **Nothing in the teacher UI links
  to it**, and it is not a mode inside the teacher dashboard.
- **The admin account is not also a teacher account.** One person, two
  identities, so ordinary work cannot wander onto an admin surface.
- Admin writes go through the **service role with an audit row**, never a
  teacher's RLS context — you want to know what changed the free tier and when,
  even when only one person could have done it.

---

## Settled 1 Sep

**One class during the trial.** Paying unlocks more.

**The seat cap is per activity per school year: 30 STUDENTS, not 30 uses**, and
they need not be members of any class. *Use it as often as you like, with up to
30 children a year.* Metering uses would be renting, which `/pricing` says this
is not, and would punish a teacher for running the same activity twice with the
same class — which is ordinary teaching. That is a better meter than a class
limit — it does not care how a teacher organises children, and it survives a
teacher who runs the same activity twice with different groups.

**Student 31 is refused.** The number is the number. Note where that failure
lands: on a child, mid-lesson, arbitrarily whichever one is 31st through the
door. **So the teacher has to see the count climbing well before it binds** — a
cap that is only announced by refusing a child is a cap that will be discovered
in front of a class.

**One trial per teacher, ever.** A second purchase adds its activity to the
account and grants no new month. Otherwise the trial is farmable, and a second
month converts nobody the first did not.

**Guests play Signal Check and nothing else.** Cases and Operations require an
Agent account. A live guest picks a name for the game and it goes when the game
does — that is `lib/live/identify.ts`, a run row with a `nickname` and **no
`agents` row at all**, which is what Signal Check has always used.

**This is what makes the seat cap countable.** Everything metered now requires a
stable identity, so 30 students means 30 students. The earlier collision — a
guest rejoining and spending a second seat — stops existing rather than being
managed.

**Signal Check is free tier, so nothing about it is capped**, and the limit that
matters there is how many *play*, not how many arrive.

### Lapsing never deletes anything

**Entitlement is computed at request time. Lapsing changes zero rows.** The
server asks what a teacher is entitled to now and permits or refuses; it does not
archive, flag or move content. So there is no cascade to get wrong, nothing to
restore on resubscribing, and no code path that could delete a child's progress
for a billing reason.

**This is not a preference, it is the difference between a paused class and a
deleted one.** `deployments.class_id` is `on delete cascade`, and
`phase_progress`, `task_progress`, `attempt_log` and `agent_selections` all
cascade from `deployments`. *Set `archived_at` on lapse* looks just as reasonable
and puts a delete-shaped operation on the billing path.

**Lapsing reverts to activity level with no classes** — earlier drafts kept one,
which is superseded. **The paused classes stay visible, read-only**, with one
line saying they are paused and resubscribing restores them. Hiding them
produces the exact impression — *it deleted my data* — that all of this exists to
avoid, and an empty dashboard produces it fastest.

**Signal Check is free tier.** It is what the homepage sells, so it has to be
reachable by someone who has bought nothing.

**Work made during the trial stays visible. Capacity is what is withdrawn, not
access.** *(Recommended, awaiting confirmation.)* Three reasons: it is a record
of what children did, not merely a feature, and hiding it has a safeguarding
flavour; a month of real results is the strongest argument for subscribing, so
concealing it removes the evidence; and teachers show this to a head of
department, so something that was on screen in a meeting and later vanished
reads as *it deleted my data*, whatever the billing page says. **Keep what you
made, lose the capacity to make more.**

## Open — not yet decided

**The school year turns over on a declared month**, held on `schools` so one
school has one year — with a fallback on `teachers`, because `school_id` is
nullable and schoolless teachers exist from before signup required one.

## Open — not yet decided

1. **Does the count include a child who joined and never started?** A register
   of 30 holding three who opened the page and closed it binds early and reads
   as broken. The cap is on who *plays*, so the count probably should be too.
2. **Nothing writes an entitlement yet.** The redemption flow is the first
   build, and it needs no further decisions: one code per activity, one trial
   per teacher ever, grant on redeem.

## Dormant, on purpose

**`agents.is_guest`, its two check constraints and the reserved `GUEST-` prefix
were built 1 Sep for guest access to activities, then superseded the same day**
when Cases and Operations became account-only. Live guests never used any of it —
they are run rows with a nickname. **Being removed by a forward migration, not a
rollback.** Recorded here so nobody finds an orphan column and either deletes
something load-bearing or builds on something abandoned.
