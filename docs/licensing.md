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

**A flag on the teacher row is acceptable; a shared session is not.** If
`is_admin` sits on `teachers` and admin routes only check it, the teacher's
session cookie **is** the admin credential — so any weakness in teacher auth
reaches the control that sets the free tier for every school.

- `/admin/*` checked server-side per request. **Not a mode inside the teacher
  dashboard**, and nothing in the teacher UI links to it.
- **Re-enter the password to enter `/admin`**, so a stolen session is not admin.
- Admin writes go through the **service role with an audit row**, never through
  a teacher's RLS context. The schema is scoped by `teacher_id` throughout; a
  session that reads across all teachers makes every later policy harder to
  reason about.

---

## Open — not yet decided

1. **One trial per teacher, or one per code?** Three purchases should probably
   not buy three months.
2. **What is "one class"?** Concurrent, or per year? A teacher running the same
   activity with two classes at once — blocked, or allowed?
3. **What happens at student 31?** Refused, or admitted with a warning to the
   teacher? Refusing a child mid-lesson is the worse failure.
4. **Is Signal Check in the free tier?** It is the headline feature and the
   thing the homepage sells.
5. **What happens to work made during the trial month?** Banks authored,
   classes built, progress recorded. **Taking those away at day 31 would be
   hostile and should not happen by accident** — decide it deliberately.
