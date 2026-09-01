# Brain Bureau — STATUS

The coordination layer between parallel Claude Code sessions and the Claude
chat conversations used for project management. It records what is moving
*now*; permanent traps and conventions are in `CLAUDE.md`.

**Read this file before reporting on it.** Four claims in one day that §4
lacked things §4 contained, all from a session that had not opened it. Cite by
quoting the text, never the section number — numbers move.

**Not a record of environment or deployment state** — that rots silently. Every
claim names the command producing it; **[verify]** means nobody has checked. If
this file and the repo disagree, **the repo wins.**

**Public**, mirrored to
[thebrainbureau-docs](https://github.com/ThePYPGuy/thebrainbureau-docs) by
`npm run docs:sync` — a mirror, never a second source of truth.

**Provenance.** §2–6 rest on commands run 2026-08-25: `doctor`, `deploy:check`
(local and `--prod`), `git log`/`status`/`worktree list`, `package.json` read
directly. Stage progress and the key rotation come from the sessions’ own
reports — attribution, not verification. Re-checked 2026-08-31: `doctor`,
`git worktree list`, `git rev-list`, `package.json` and `scripts/` read
directly. **§2's two planned builds are scope from Maciej's brief, not code**
— neither has anything in the repo yet to verify against.

§7’s font findings are the exception: measured in a running browser this session,
not reported. Rendered DOM width against a deliberately nonexistent family —
the only method that gave a true answer, and the reason §10 now says every
cheaper check is wrong.

## 1. Sessions

| Name | Owns | Worktree | Holding the tree? |
|---|---|---|---|
| **Operation Builder** | Prime Directive + the Case File skin | own, on `case-file` + PD | n/a |
| **Website Infrastructure** | Platform, engine, scripts, checks | own, on `platform` | n/a |
| **Quiz Maker** | Signal Check and the live modes | own, on `signal-check` | n/a |
| **Website Designer** | The Bureau face — marketing site, both dashboards | **`main`'s tree** | yes |
| **Doc Manager** | `STATUS.md`, `CLAUDE.md`, `docs/`. Not `scripts/docs-sync.mjs`, and **never `git push`** — publishing is the builder's whose code goes out | own, on `docs` | n/a — never in main's tree |

**Before your first write, run `git status --porcelain`.**

- **Empty** — the tree is yours. Work, then commit only the paths you
  authored, by name.
- **Not empty** — stop. Do not edit, commit, stash, renormalise or switch
  branches. Say which files you found, and wait for the tree to clear.

Say so when you commit. Four collisions on 2026-08-25, two destroying work;
mechanics in `CLAUDE.md`.

## 2. Active work

**Only one row may show uncommitted files at a time**; §1 says how to check.

| Stream | Branch | Uncommitted | Status |
|---|---|---|---|
| Case File skin | `main` | none | Stages 1–2 done; 3 blocked on images — dormant |
| Website redesign | `main` | stray `:Zone.Identifier` | **all three phases live** — no surface left on the old chrome |
| Prime Directive | `operation-prime-directive` | none | **identical to `main`** at `cac3f44` |
| Platform | `platform` in `../tbb-platform` | none | scoping and importer both merged to `main` |
| Docs | `docs` | `STATUS.md`, `CLAUDE.md` | own worktree; merges to `main` `--ff-only` |

**Doc commits ride to `origin` on the next code push** — `docs:sync` publishes the
mirror, not `origin`. The cycle, not a backlog, and not another session's to push.

## 3. Overlap risks — READ BEFORE ASSIGNING WORK

Live collisions only; standing hazards are in `CLAUDE.md`.

- **A branch merge is a snapshot, not a subscription.** `operation-prime-directive`
  is 15 behind. Run `git rev-list --left-right --count main...HEAD` before
  judging how anything on a branch looks — never the last merge date.
- **`"import"` globs now; `"import:training"` still does not.** `package.json:16`
  names three bank files by hand, so the bank refactor and every new bank collide
  there the way activities used to. Read both lines before assuming either —
  `"import"` runs `scripts/import-all.ts`, which walks both directories.
- **`bb49a62` duplicates `5fda3d3`** — verified; cherry-pick residue is inference. Likely to conflict on merge.

## 4. Publish state

Pushing to `main` deploys the app. **Migrations do not run themselves. Content
does not import itself.** Both fail silently.

**Live at https://thebrainbureau.app** — the apex 308-redirects to `www`, so a
bare 200-or-fail check reads a healthy site as down. Follow redirects, or check
the `www` host.

**Codes are open on production and nobody has played through one**, so far as the
repo can tell. §10's rows about what happens to a child stop being hypothetical
the moment one does. Read live codes from the dashboard, not from here.

**Only the main worktree can reach production.** `--prod` elsewhere fails with
*no linked Supabase project found*, which reads like a broken install and is a
boundary: you cannot write to production from a branch by accident. Run
production commands from the main worktree. (Why, and the override, in §5's
file.)

**What is proven is the database, not the running code.** `--prod` never asks
which commit is serving. **Fingerprint the served CSS** — a bundle holding
`crtViewport` but no `.loginPanel` proves that commit is live, with no write.

**`--prod` was clean at `26bcafe`**, 1 Sep: **31 migrations**, all in step.
**Re-run it; do not read this as current.** `?` is not `DRIFT` — it means
unconfirmable, and the answer is usually a re-import.

**Sentry covers the whole app**, not only Signal Check — so the first unfamiliar
error from an old surface is newly visible rather than new.

The `completion` gate is **verified locally, not on production** *[verify — none
since 25 Aug]*: join a real code, watch the ending stay absent until the last lock.

**Adding: migrate, import, re-check, push — the check gates the push. Removing:
push first**, because the running code is the old code and still selects what you
are dropping. `CLAUDE.md` carries the rule.

**Before any push** run `git diff --stat origin/main..main --
supabase/migrations/ content/`. Neither deploys itself, and both fail silently.
**Migration 31 was the *safe in either order* case** — it replaces a function both
versions call, `p_points` defaults, so applied first or deployed first nothing
breaks. **The build then failed on a missing `@types/pngjs`**, which
`deploy:check` cannot see: it reads the database, and the database was already
correct. **`--prod` green while production serves a three-hour-old build is a
state this project can reach**, and did.

**A range with more than one builder's code is Maciej's to push.** §1 names *the
builder whose code goes out*, which answers nothing with two sessions in it —
and only `main`'s tree reaches production, which is the tree WD works in.

**A delegation Doc Manager records cannot authorise a push**, and both builders
said so independently: §4 carries it only because Doc Manager's commit put it
there, and every session commits as `ThePYPGuy`. **Verified work builds and waits
for Maciej's own word.** `CLAUDE.md` has the rule.

## 5. Environment

`npm run doctor` — 2026-08-31: **all checks passed**, no warnings. Both former
warnings are closed: `uv` is symlinked into `/usr/local/bin`, so a plain
`bash -c` finds it and not only a login shell, and the image-gen API key
resolves. This is the first fully clean run recorded here.

Paths, worktrees, project refs and Maciej's folders are in
`docs/local/environment.md`; `git worktree list` and `doctor` outrank it.
`.env.local` points at **local** Supabase and takes the *local* key —
production keys belong in Vercel only. What `doctor` does *not* assert is in
`CLAUDE.md`.

## 6. What is actually built

Verified against `package.json` directly, not against a summary of it.

| Check | Command | State |
|---|---|---|
| Preflight | `npm run doctor` | built (`d446fb4`) |
| Publish drift | `npm run deploy:check [-- --prod]`, `import:one` | built (`0760a88`); `--prod` fixed (`8839d38`) — exits rather than checking the wrong database. `import:one` sends one drifted file |
| Line endings | `.gitattributes` | built (`f0563c2`) |
| Skin variety | `npm run skins` | built — counts authored files and names its scope |
| Tests | `npm run test`, `npm run e2e` | vitest suite; 6 e2e scripts, incl. answer leak |
| Importer safety | `npm run test:reimport` | built |
| Targeted | `test:dashboard`, `:signup`, `:school`, `:entitlements`, `:curriculum` | built, 5 scripts |
| Bank checks | `test:columns`, `:insight`, `:images`, `:search`, `:edit-after-serve`, `:copy`, `check:alt`, `check:keys`, `check:tokens` | built, 9 scripts; `check:tokens` fails on a hex or font literal outside a token block |

**Activity schema is locked at 0.4**, and `activity-schema-v0.4.md` now exists —
written 31 Aug, corrected on the way in. **`npm run test` is green — 395/395, 18 files**, run on `signal-check`
31 Aug. Still no CI, so that number ages the moment anybody commits.

## 7. Recently completed

**A window, not a record** — oldest entries fall off, and git holds the rest.
This is the section that pays for the 250-line cap; §8 and §10 do not. Hashes
from `git log`; descriptions are each session's account of its own work.

- `1b01e1b`, `ec40653` — **the teacher-facing copy is data, and Signal Check got its quadrants.** The block is `curriculum.marketing`, nested in an existing `jsonb` — **an `activities.marketing` column would have cost the same migration and the same two column lists** as the per-phase field it was chosen to avoid. The import refuses on count, on an unknown slug, on a blank line **and on order**: two lines swapped pass a count and a set check while the page describes phase three under phase two. Broken four ways to prove it, each refusing before a row was written. *(WI.)* Plus the projector grid keyed to the canonical index and phone type 17px → 31.5px. *(QM.)*
- `ac0bcf2` — **the sidebar splits by what a thing IS, not by who owns it.** Bureau Library goes as a top-level entry and **costs nothing** — read from the database, all three Bureau banks are `public`/`published`, which is exactly what `?scope=public` already selected. **Classes could not be fixed by repointing** because there was no list of classes: `/dashboard/classes` is new, and *Your classes* on `/dashboard` became **Assigned work**, since it was competing with a real page for the same word. Cases and Operations point at `/missions#cases` and `#operations` — **true today and replaced by step 4 rather than contradicted by it**. All ten entries clicked, not read. *(WD.)*
- `9ae8f62` — **one frame token and nine type steps, where there were a hundred and eight.** `--bb-max` becomes `clamp(1160px, 92vw, 1600px)` — side margin at 1920 falls from 380px to 160px — and **prose is untouched**, because the reading measures are their own caps. Nine clamped type steps now carry **all 121 declarations** that were set in place, including thirteen in `page.module.css` that my brief never named and which were why the homepage kept 9px labels. The dashboard rendered **fifteen sizes from 9px to 34px**; it renders seven, none below 13px at 1440. Every `var()` keeps its old literal as a fallback and the tokens live on `.surface`, not `:root`, **so nothing here can reach an activity** — confirmed at 0 pixels on all four activity surfaces. *(WD.)*
- **Queue 1 shipped and is live** — `/join` rebuilt on one identity question, the homepage static behind `proxy.ts`, loading states in both faces, Signal Check's four playtest faults, speed-scaled points with a grace band and a top-three podium, the visual regression harness, and migration 31. Deployed 1 Sep at `26bcafe`; `/` went from `private, no-cache` MISS to **`public` HIT**. The lessons are in `CLAUDE.md`; git holds the rest.

## 8. Next up

1. **Redemption: one `/redeem`, not a second signup.** Activities carry only a
   `slug`, so a code needs a home, and the route must serve signed-in and
   signed-out alike — the PDF link cannot change. **The only irreversible piece,
   so it starts with Maciej present rather than from a peer's brief** — four
   briefs were wrong about the repo on 1 Sep and each was cheap only because the
   work was reversible. *(WI: code, route, write. WD: the page.)*
1. **`app/brand.module.css` has zero importers**, but `check-tokens.ts` grandfathers
   six literals by it — deleting it changes what the checker enforces. *(WI.)*
1. **`_note` says `prefix` is never rendered; `Tasks.tsx:428` draws it.** *(Op Builder.)*
1. **Queue 2, in order** — `docs/local/briefs/queue-2.md`, with
   `ui-pass-2.md` behind it. **1** layout pass *(WD)* · **2** sidebar and top bar
   *(WD)* · **3** Signal Check phone type and projector grid *(QM)* · **4** Cases
   and Operations pages *(WD)* · **5** question maker · **6** prices and purchase
   routes *(WI, **with Maciej** — the only irreversible step)*. **1 and 3 run in
   parallel**; 2 waits on 1 because it is the same session and the same files.
1. **Write the teacher-facing phase lines** — Zero Hour 5, Prime Directive 7.
   Only Global Intel Cards was ever written out in full. *(Maciej.)*
1. **Re-establish that the redesign never touched activity chrome.** Baselines
   are all dated 1 Sep, so they prove nothing about before. Capture a
   pre-redesign commit in a temporary worktree and diff.
1. **Run `sudo npx playwright install-deps chromium` once.** Not housekeeping:
   without it `check:visual` cannot run, so **a session cannot gate its own
   work** — WD had to substitute a before/after capture for the real check. And
   **baselines cannot be made from Windows**: an unchanged page differs by
   326,418 pixels across platforms, so today only WI can regenerate them.
   *(Maciej.)*

## 9. Open decisions — waiting on Maciej

1. **Two open questions on licensing** — `docs/licensing.md` has them, and
   neither blocks the redemption flow, which is the first build. **An open
   deployment code admits anyone and always has** — `resolveAgent` creates an
   agent for any unused codename. Parked under licensing: entitlements change
   what a code should guard.

## 10. Known silent failures

Open items. Standing traps are in `CLAUDE.md`; prune a row only when it is
closed there or here, and **never to hit a line count** — that emptied this
table once. **Documentation does not fire at 11pm.**

| Failure | Symptom | Owner |
|---|---|---|
| Short numeric answers are not leak-checked, and cannot be | `test-answer-leak.ts` derives forbidden values from each activity's own `answer` fields, `_`-prefixed subtrees and `completion` — but with two floors that **are** its coverage. `PROSE_FLOOR` 12 chars, below which a secret string is usually an identifier: `_evidenceDesign.column` is "colour", public by design, and forbidding it would fail on correct content. `DIGITS_FLOOR` 3, below which a figure cannot be told from any other on the page — Lock 06's answer is 2 and appears inside its own exhibit, correctly, because it is the sum. Lock 07 is covered only because its `prefix` makes the typed value `C-09` | **stated, not fixable** — written out in the script so it is never cited for more than it does |
| A check that resolves a tool by bare name | It reports the launching shell's `PATH` as the machine's state. `uv` is installed and `doctor` passes interactively and fails under `bash -c`, because `~/.local/bin` comes from `~/.profile`. Reported as *not installed*, which is a different problem with a different fix | **fixed** `84138c5` — it now names the path it found and says which of the two it means |
| A check that fires on every call is one nobody reads | The aspect-ratio check first compared reduced fractions and flagged a genuine 4:3 response of 1200×896 — 75:56, 0.45% out. Decimals within 2% now, tested against six cases including that one. Same lesson `doctor` already carries about its anchored `/images/` pattern | **fixed** in the art skill |
| Sentry stored city-level location and a timezone, and the client could strip only one | `user.geo` is derived from the connecting IP **at ingest**; `contexts.culture` — locale, calendar, timezone — was collected in the browser. Two fields, two different fixes, and *stripped at the client* could only ever promise the second | **`culture` fixed**, verified on a production event: Additional Context now holds `react` and `trace` only, with `trace_id` intact and frames still mapped. **IP storage turned off 31 Aug — unverified**, because the newest event predates the change and an old event cannot answer it. The next one will |
| **The accessibility override does nothing on a Bureau surface, and is never even switched on there** | `applyClearView` is called only by `/live/*` and `/join`, so **`data-clear-view` is never set on a Bureau page at all** — one layer beneath the token problem. `globals.css:1981` redefines **skin** tokens — `--surface`, `--ground`, `--ink`, `--edge`, `--skin-font-body`. The Bureau face reads `--bb-*` and it touches none of them, so *easier to read* changes nothing on `/profile`, the dashboard or the marketing site. **Not a regression** — the old `.shell` never responded either — and it works on every Field surface. But it now sits **visibly on a child's own page doing nothing**, which is worse than not being there | **open** · needs a high-contrast Bureau palette, so a build rather than a fix |
| **Nothing on the platform is rate limited** | Not one route: not `/api/join`, not `/api/agent/login`, not `/api/agent/exists` — which is public and unauthenticated by necessity, because a child has no session when they type a codename. A four-digit PIN against a known codename is ten thousand guesses and nothing counts them. No incident; the platform has almost no users. **It stops being theoretical on the day one class uses it** | **open** · unowned — a platform-wide posture, not one route's bug |
| **A phone joining a game got a light border down both edges** | `globals.css:1710` sets `body { padding: 10px 6px }` under 640px, next to `.monitorFrame` — it is chrome for the CRT bezel. On a full-bleed page like `/live/join` there is no bezel, so the padding shows the page ground through it. Reads as a design choice rather than a fault, on the surface a child actually uses, and no contrast or pixel check looks at `body` padding | **fixed** `a71a2cd` — by **deleting** the rule, not moving it: `.crtViewport` (`globals.css:1068`) already draws the monitor's gap at `30px 16px`, so `body` was laying a second one over it on every page |
| **The Field face is the default for every Bureau element** | `globals.css` styles bare `h2`/`h3`/`h4`/`p`/`strong`/`button`/`input` for the CRT *and* gives those variables real values in `:root` — `--body-text: #d0d0d0`. So a bare `<p>` on a cream page renders at **1.40:1**, and the hero once rendered at 1.99:1; both screenshotted fine and were found only by measuring. Website Designer neutralised it inside `.surface` rather than editing a file it does not own. The fix is scoping those selectors at source, and **every form in phases 2 and 3 meets this** | **fixed**, in `main` at `5bfbfc7` · `:where([data-skin], .zoomBackdrop)` adds no specificity, so nothing that beat these stops beating them |
| **A diagnostic read outside the question it was built for** | `content-fingerprint.ts` answers *did anything other than the named file move during this import*, where ownership is irrelevant — so it never selected `owner_teacher_id`. Its output was then read as an inventory of platform content, where ownership is the only thing that matters. **Every value it printed was correct.** A slugless `training_sims` row was called an orphan and queued for deletion; it is Maciej's own draft quiz, made in the app on 25 August. Verifying it against production repeated the same omission and made the wrong claim more credible | **fixed** — the output now marks every row `platform` or `teacher <id>`, so that reading is refused rather than available |
| `deploy:check` read only one direction | It compared repo files to the database and stopped. `checkActivities` already asked whether a row had no file behind it; `checkTraining` never did | **fixed** — one helper, unmatched rows report `?` rather than DRIFT, since DRIFT's remedy is *run the importer* and importing never removes a row |
| A guard that closed the only recovery path | `alreadyDone` skipped `settleCompletion` whenever the task was already correct — protecting against a double-award the callee already refused, and in doing so shutting the door a stranded child would push on. **Fixed**, and the state is now repaired on load | **self-healing, not closed** — the two writes are still not atomic and cannot be made so from the client; it needs a Postgres function, and `check.ts` says so where it happens |
| A repair that reads as a loss | The reconcile awards Intel, but `loadStudentState` had already read the agent row — so the phase opened with the old total beside it and the award looked like it had gone missing. Caught in testing; the agent is re-read only when something was repaired | **fixed** — a silent repair still has to be visible where it lands |
| A failed import left rows it created | Positions were restored and creations were not — no transaction, so a run that died partway left rows nothing accounted for and no error naming them | **fixed** — one transaction, migrations 16–18, verified by forcing a failure midway |
| A test that does not reproduce the reported bug | The first recovery test poisoned the *last* phase, so the failure landed after all seven had been renumbered: it caught the ordering corruption and stranded nothing, and would have passed against the broken importer for the wrong reason. Moving the fault to the first phase reproduced it exactly | prove the test fails against the original bug, not a neighbour |
| Leak guards that cover only the activities they were written for | `e2e` passes while an untested activity ships its whole deduction. Prime Directive's served state is checked by hand, and it is the one that already shipped an answer in a caption | **fixed** — `scripts/test-answer-leak.ts` reads the directory and derives each forbidden value from the file, so a new activity is covered by construction rather than by being remembered |
| One activity's fiction hardcoded for all | Every activity shows "⚠ ZERO HOUR" and completes on "VAULT SECURE". A workshop short-circuit case ends by securing the Value Vault, and it renders perfectly | **fixed** — `Mission.tsx` takes `expiredLabel` and `doneLabel` from the theme, defaulting to `⏱ TIME UP` and `COMPLETE`. The Zero Hour literals survive only in a comment recording why |
| `npm run skins` counted the database, not the content files | It reported an archetype count from rows, so the file you were authoring — not imported yet — did not count, and a row whose file was gone still did. Wrong in the one moment the tool is used | **fixed** `2b6df67` — it reads `content/activities` |
| New tables needed an explicit `service_role` grant | Without it reads failed as a permission error that reads like a missing row — the worst disguise, since it sends you hunting for data that is already there | **fixed** — `deploy:check` audits every table for the grant |
| **The importer's column lists live in two languages and nothing compares them** | `import-activity.ts` builds the payload; the `import_activity()` SQL function consumes it with explicit `INSERT` lists. Add a column, author it, and it is dropped silently — then `content_hash` is taken from the *file*, so the hash moves and `deploy:check` reports the database matches the repo while the column sits empty. Written 31 Aug, so it is the least-exercised code in the repo | **fixed** — `npm run test:columns`, three checks: schema against the SQL function, payload against it, and a real round trip. It reads the *installed* function with `pg_get_functiondef` rather than the migration file, so what is checked is what is running |
| **Nothing verified that the drawn things rendered** | No script mentions `hotspot`, `facsimile`, `phaseIcon` or `data-icon`; sixteen hotspots across twelve plate forms and seven glyphs were checked by hand, once, by one session. Rename `.plateRows` or change a `data-icon` and `doctor`, `e2e`, `tsc` and 131 unit tests all still pass. The page renders — it renders *wrong* | **§8's visual regression check** is the only cover this work will ever have |
| **A check that agrees with the bug it was written to catch** | Both fixes this session shipped with a check that passed for the wrong reason, each found only by deliberately breaking the thing it guarded. The import fixture passed against the *broken* importer until the fault was moved. The grant audit read a `pg_catalog` view that hides exactly the tables it was hunting — so it found nothing and reported clean, which is indistinguishable from finding nothing because nothing is wrong | **pattern, not a defect** — a guard is only proven by breaking its subject, and both were |
| **Signal Check's reveal is off by one, and it is live** | `revealFor` does `this.questions[position]` — a 0-based array indexed by a 1-based position — so the board shows the **next** question's answer, explanation and option labels. **The reported mis-scoring was never real** — that came from a harness reading canonical option order and skipping the per-player permutation. The header reads *QUESTION 2 OF 22* for the first question served. Found while staging a screenshot, isolated to one player and one answer, and verified against `session_questions` | **fixed and deployed** `9eb2071`. The constructor now refuses anything not renumbered, so the fault cannot return silently — it raises before a child joins |
| **The scrubber redacts the debug id, so source maps never resolve** | `UUID_ANYWHERE` in `lib/live/telemetry.ts` matches Sentry's own `debug_meta.images[].debug_id`, which is a UUID by design. Sentry discards it — *invalid debug identifier* — and with nothing joining the served bundle to the uploaded map, every frame arrives minified. **The maps upload correctly and are never applied.** The earlier instance of this ate `trace_id` and `public_key` and was found because it caused a 400; this one produces no error at all | **fixed** `87a737e` — the scrubber now names the fields that can carry what a child wrote and leaves the envelope alone. A production payload carries an intact debug id matching an uploaded map, so *invalid debug identifier* cannot recur. **A mapped frame is still unseen** |
| **Telemetry carries an identifier and nothing errors** | Sentry payloads are scrubbed at the client, so a regression in the scrubber sends codenames, guest nicknames or a game PIN to a third party while looking exactly like correct behaviour. Needs a guard that spells out its own patterns — one importing the scrubber's is a single gate, and empties when the scrubber breaks. `docs/identity-and-access.md` has the contract | **candidate** · Quiz Maker — before the first classroom session |
| `tracesSampleRate` is 1.0 | Measured: roughly **140 transactions** for a thirty-player session of three questions. Left untuned on purpose so the figure came from the harness rather than from anyone's estimate | **measured** — tune when a real session shape is known |
| **Throughput, not monthly volume, is the binding Realtime limit** | Measured 31 Aug against thirty real sockets: **96 msg/s peak**, 3.6 deliveries per player per question, 381 for a session. Supabase counts each delivery to each client, so one broadcast into a room of thirty is thirty events — which is why a monthly total was the wrong thing to worry about. Still does not error at the ceiling; it stops delivering | **measured**, and resolved by moving to Pro: 96 of 500 |
| **A session desyncs and one player's score diverges from the server's** | Client and server both hold a score. They can drift apart with nothing comparing them, and the child sees only their own number — so it reads as correct until the review screen, or until a teacher is asked why two agents with the same answers finished apart | **candidate** · Quiz Maker |
| Pruning a list by its length rather than its contents | This table was emptied over one session. Every removal was defensible alone — fixed, or recorded in `CLAUDE.md` — and nothing checked whether the section still said anything, because the number being watched was the line count | **fixed by rebuilding** — prune against what is still open, never against a budget |
