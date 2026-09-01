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

## Prices — decided 1 Sep

**A Case is $5. An Operation is $3.** Provisional, and meant to be changed.

**Price is per activity TYPE, not per activity.** `activities.activity_type`
already carries a CHECK over `operation`, `case`, `training`, `fieldOp`, so the
set is closed and a map over it cannot silently miss one.

**The activity page shows its own price and links to `/pricing` for the
subscription.** Subscription figures live in exactly one place, which is what
`/pricing` already says about itself.

**One function, `priceFor(type)`, and nothing else knows where the number comes
from.** Today it returns a constant. When `/admin` exists it reads a table
instead and no caller changes — the same move the free tier needs, and for the
same reason: both are commercial config that must change without a deploy.

**Three things that are cheap now and expensive later:**

- **Integers in minor units.** `500`, not `5.00`. Money in floats is a classic
  and it is free to avoid on day one.
- **State the currency.** `$` is ambiguous; TPT settles in USD, so say USD
  rather than leaving it to be inferred.
- **Record what was paid on the entitlement, at purchase.** A price is a lookup
  *today* and a fact *forever*. If `entitlements` does not carry the amount, then
  raising a Case to $6 silently rewrites every receipt that ever said $5 —
  including for a teacher asking their school to reimburse it.

## Measuring the funnel — decided 1 Sep

**Analytics runs on teacher and visitor surfaces only** — *not* on the seven a
child uses: `/join`, `/terminal`, `/training`, `/live/join`, `/live/host`,
`/live/play`, `/profile`.

**"Bureau face" is the wrong test and an earlier draft of this line used it.**
`/profile` wears the Bureau face deliberately and renders the same header and
footer as the marketing pages — so one line in `SiteFooter` would have covered
the marketing site *and* a child's assignments page, while looking like the
tidiest possible fix. **The axis is who uses a page, not which skin it wears.**

**Mounted page by page** (`2997224`, `bfc601b`), not below a shared layout: there
is no layout between the root and the marketing pages, and the structural answer —
a route group — would move seven of Website Designer's files mid-work. **The
trade was chosen for its failure mode:** a forgotten Bureau page is
under-measured, where a path list in the root layout would silently give a new
child page a third party.

**Verified by served HTML, not by the visual check** — `@vercel/analytics` renders
nothing, so `check:visual` was green and could not have caught this. `/profile`
and `/terminal` were re-checked **with a session**, because signed out they merely
redirect and a clean result would have proved nothing.

**What the script sends, measured 1 Sep.** The SDK composes exactly `{ o: page
URL, sv, sdkn, sdkv, ts }` and posts it to `/_vercel/insights/view`. **No
identifier, no cookie, no user id, no referrer** — the page URL is essentially the
whole content, plus version strings and a clock. **Child pages send nothing**:
zero analytics requests and zero off-host requests of any kind.

**In production no third-party domain is contacted at all.** Dev pulls the script
from `va.vercel-scripts.com`, which is a genuine third-party origin and looks
alarming in a network log; **that is a dev artefact.** Production serves
`/_vercel/insights/script.js` same-origin, so script and beacon are both ours and
Vercel receives the data **because it is the host, not because a third party is
dialled.**

### But Vercel observes every request anyway, and that is a different question

**Yes — at the hosting layer, independent of the script.** Every request to every
page, child pages included, reaches Vercel's edge and is assigned an
`X-Vercel-Id`. The analytics *product* does not run there and was verified not
to. **Vercel as host observes regardless, because it is the thing serving the
page.**

So mounting page by page does exactly what it was meant to and **nothing a
component tree could ever do more of.** It controls the product; it cannot
control the host.

**Two sentences, and only one of them is true:**

- *"No third-party script runs on the children's screens."* — **true, measured.**
- *"No third party receives data about the children."* — **not true in the way a
  head teacher means it.** Vercel processes every request a pupil's browser
  makes: URL, IP, user agent, timing, for `/terminal` exactly as for `/pricing`.

**That is a processor relationship, and what answers it is a contract rather than
a component.** Which sentence to use is Maciej's decision and it is worth taking
properly — a school's data officer will ask, and the honest answer is available
now rather than under pressure. *(Doc Manager is not the right source of advice
on what the obligation actually is.)*

**Also true and worth knowing before being asked:**
`/_vercel/insights/script.js` answers 200 on production whether or not any page
references it, so Web Analytics is enabled at project level. Serving an endpoint
is not collecting from it — nothing on a child page loads it — but a school
reading the network tab would find the URL reachable on the domain.

**Not measured, and it matters:** nobody has observed a real transmission. Dev
disables sending (*"Debug mode is enabled by default in development. No requests
will be sent"*), and production does not yet carry the analytics commit. **The
payload shape is from the SDK's own debug output and its source — good evidence,
not a captured request.** Nor is it known what Vercel *retains* from hosting:
`X-Vercel-Id` shows a request is traced, not what is kept.

**Not squeamishness — the questions are not there.** Everything worth knowing
about a child-facing surface is already in the database and answered better:
whether a signed-up teacher ever assigns anything, whether an assignment is ever
played, **which phase children stop at**, whether a teacher returns weeks later.
Those are outcomes; a pageview is not. Adding a third party to a child's screen
to learn less than a query already tells you is a bad trade at any privacy cost.

**And this project has been wrong about that before.** The Sentry contract says
identifiers are stripped at the client — then records that `user.geo` at city
level and `contexts.culture` including timezone arrive anyway, and that *stripped
at the client* was an incomplete frame. Second third party, same caution.

**What analytics does answer, and nothing else can:** the pre-account surfaces.
Where a visitor arrives, which page they leave from, whether TPT traffic
converts, and **which Cases and Operations draw interest** — including from
people who will never appear in the database because they never sign up.

**What it will not answer, and what to build when it matters:** *which teacher
looked and did not buy.* Analytics gives counts, not people; the database records
purchases and assignments but **never records that an activity page was viewed**.
One row — teacher, activity, timestamp — turns *40 views* into *eleven teachers
looked and two bought*, which is the number that says whether the price or the
preview is wrong. **Worth building after the purchase routes ship, not before:**
it is meaningless without sales to attribute, and it should be shaped by the
question it is finally asked.

## What is recorded about a child, since the question gets asked

Mostly pseudonymous: codename, PIN hash, Intel, clearance, answers, elapsed
times, progress. **One field is not.** `roster_entries.real_name` is `not null`,
one per agent — a teacher fills it in so a class list is readable, and progress
links to it through `agent_id`. **A school will ask; the answer is that you hold
one real name per pupil and everything else hangs off a codename.**

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

**Already enforced, not merely intended** —
`20260824000012_deployment_requires_entitlement.sql` splits the deployments RLS
policy by operation: `select` returns a teacher's deployments *including any
whose activity they no longer hold*; `update` keeps working when a subscription
lapses, because *"locking that would trap a teacher with a live join code they
cannot turn off"*; only `insert` requires an entitlement. The application check
in `actions.ts` stays as the clear message, with the policy as the backstop.

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

## What must exist before the first TPT listing

**Only one thing is irreversible: the code in the download.** A published PDF
cannot be recalled, so the code's format must be final and redemption must work
*before* anything is listed — otherwise the first buyers hold a string that does
nothing, and no server-side dial fixes that.

That means: somewhere to store a per-activity code (`activities` carries a `slug`
and no code today), the format settled, and a redemption route that writes an
`entitlements` row.

**Everything else can follow a first sale, and can be applied retroactively.**

| Missing | Why it can wait |
|---|---|
| Trial expiry | `entitlements.granted_at` already records when. Expiry can be computed later and applied correctly to rows granted before it existed. |
| Seat counting | Reconstructible from `task_progress` / `phase_progress` — `agent_id`, `deployment_id` and timestamps are all recorded. |
| `schools.year_start_month` | Only binds when a cap does, which is 30 students in. |
| Free tier as data | `lib/entitlements.ts` holds a development policy keyed on email domain, and **fails closed** — its own test names the danger: *no entitlements read as unrestricted*. |
| Admin dashboard | Nothing depends on it. |

**The one thing to avoid is selling before expiry exists and forgetting.** The
trial would not end, and early buyers would hold Full Access indefinitely.
`granted_at` makes that recoverable, but clawing back access someone has had for
months is a conversation, not a migration.

## Dormant, on purpose

**`agents.is_guest`, its two check constraints and the reserved `GUEST-` prefix
were built 1 Sep for guest access to activities, superseded the same day** when
Cases and Operations became account-only, and **removed by migration 30** — a
forward migration, not a rollback of 29. Live guests never used any of it: they
are run rows with a nickname, and `e2e:signal-check` passed whole through the
removal, including *a guest earns nothing*.

**No child ever used the door.** Measured on production before the migration was
written, counts only: 5 agent rows, 0 with a null `pin_hash`, 0 guests, 0
`GUEST-` codenames. The `delete from agents where is_guest` in migration 30 is
for the 26 rows a local test database held — a migration that only applies on
one machine is not a migration.

**`GUEST-` is choosable again**, since the reservation went with the path. Tested
rather than assumed. Harmless today — no minter exists to collide with — but a
real behaviour change, and there is no longer any code behind the reservation if
it should ever come back.
