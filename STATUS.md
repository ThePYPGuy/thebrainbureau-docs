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
| Website redesign | `main` | stray `:Zone.Identifier` | **phase 1 and the doors live** at `d669a13` |
| Prime Directive | `operation-prime-directive` | none | **identical to `main`** at `cac3f44` |
| Platform | `platform` in `../tbb-platform` | none | scoping and importer both merged to `main` |
| Docs | `docs` | `STATUS.md`, `CLAUDE.md` | own worktree; merges to `main` `--ff-only` |

**The redesign is in `main`'s tree**, shared with the dormant Case File stream —
two sessions in one tree is the hazard, not two on one file. Projector merged.

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

**A deployment code is open on production.** `OP-35HY` → **`303616`** *[verify]*, alongside
three older codes; two classes and three agents exist. Maciej is still the only
person who has *played* anything, so far as the repo can tell — but the door is
now open, and §10's rows about what would happen to a child stop being
hypothetical the moment one uses that code.

**Only the main worktree can reach production.** `--prod` elsewhere fails with
*no linked Supabase project found*, which reads like a broken install and is a
boundary: you cannot write to production from a branch by accident. Run
production commands from the main worktree. (Why, and the override, in §5's
file.)

**What is proven is the database, not the running code.** `deploy:check --prod`
never asks Vercel which commit is serving. Confirmed *indirectly* 31 Aug:
`/api/handle?handle=…` answers `{"ok":true,…}` in production and exists only in
the `d669a13` range, so that deployment is current. The SHA itself is still
unasked — a route is evidence, not an identity.

**`--prod` was clean at `d669a13`**, 31 Aug, before the push and again after: 27
migrations, three activities, three banks, 73 skills, 20 tags. **Re-run it; do
not read this line as current.** The order that got there, and the one to reuse:
**migration, importer, re-check, then push** — the check gates the push, so a
drifted database cannot deploy. `?` is not `DRIFT`: it means unconfirmable, and
the answer is usually a re-import.

**Live:** the platform, three activities, the Case File skin, the evidence
capability and the `completion` gate. Prime Directive `bb7f9b2` imported to
production; the question bank refactor `b663ad3`. §6 carries the detail.

**Sentry covers the whole app**, not only Signal Check — so the first unfamiliar
error from an old surface is newly visible rather than new.

The gate was **verified locally, not on production** *[verify — no session has
re-checked this since 25 Aug]* — proving it there means
creating a student agent in a live database. To confirm on the site, join a
real class code and check the ending stays absent until the last lock.

**Before any push** run
`git diff --stat origin/main..main -- supabase/migrations/ content/`. Neither
deploys itself and both fail silently when missed — and **this range needs a
migration**, the first that has.

**A range with more than one builder's code is Maciej's to push.** §1 names *the
builder whose code goes out*, which answers nothing with two sessions in it —
and only `main`'s tree reaches production, which is the tree WD works in.

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

- **the codes are one namespace, and the join box is one door.** `access_codes` (migration **28, not on production**) keys on the code itself, so minting is an insert and a collision is a unique violation rather than a race two teachers can both win — and the id is chosen in TypeScript so the code is claimed *before* the row exists. **Both lifetimes are held:** a PIN leaves the registry when its session ends so the space cannot narrow, while a permanent code refuses any string ever used as a PIN — so last term's projector photo can at worst reach another live game, which is what recycling already meant, and can never reach a class. `/api/join` resolves once and dispatches: a game PIN typed into the join box is handed to `/live/join?pin=` already filled, not refused. **The router test caught a real bug before it shipped** — `normaliseCode` strips dashes, so `C-6M01` reached `isLegacyCode` as `C6M01`, and every old card would have been told *check the digits* when the digits were right. *(WI; nothing deleted, nothing pushed.)*
- `9de0d0b` — **Create exists, and Discover / Bureau Library / Create is finally the whole list.** `/dashboard/banks/new` takes either door — blank or CSV — and both land in the same editor, so the two paths converge one step in rather than forking the product. The draft state is stated on screen and Publish is disabled until there is a question to publish, which is the trap named in the brief: a bank is `status='draft'`, `session_bank_is_playable()` gates live play on `published`, and without that a teacher builds a bank and finds no *Start live* button with nothing explaining why. Deployed and verified: `/dashboard/banks/new` answers **307 to login rather than 404**, and that route did not exist before this push — a cached bundle could not produce it. **Unverified: everything behind the login.** Creating a bank against production, and `/import` landing on `?door=csv`, both need a signed-in session. *(WI.)*

## 8. Next up

1. **Delete WD's undo.** `a1da068` is in `main`, the bare rules are scoped, and
   the `currentColor` block in `site.module.css` now neutralises nothing. *(WD.)*
1. **Re-mint the two classes and four deployments** with six-digit codes from
   `access_codes`; **delete nothing**. The code is not a foreign key — progress
   hangs off `deployment_id` — so the string changes and every row lives. Ends
   with no legacy codes anywhere. *(WI. Maciej reprints and hands out.)*
1. **Prime Directive's `_note` claims the `prefix` field is never rendered.**
   `Tasks.tsx:428` and `:583` draw it now. **Delete only the "never rendered"
   and "platform gap" clauses** — keep the rest: the answer is numeric `9`
   because `validate.ts` `expectedValues()` coerces static answers to numbers
   and throws on text, a live constraint naming live code. *(Operation Builder's
   file; the fifth false sentence in that one note.)*
2. **`runs.agent_id`'s comment says guests must "appear on the leaderboard".**
   `20260831000020_training_sessions.sql`. Signal Check has no leaderboard on any
   surface — the column is right, and the sentence records a design replaced
   before it shipped. One line. *(WI's file.)*
3. **Visual regression check** — screenshot each activity, fail on change.
   **Not polish.** It is the only thing that would ever cover the drawn work:
   sixteen hotspots, twelve plate forms, seven glyphs, all verified by hand
   once. Every other check on this project passes while a page renders wrong,
   because nothing is broken — it just looks it. Doc Manager recommended
   deferring this three times and was reading it as finish rather than cover.
   **Clear intervals and cancel animations before capture** or the HUD timer
   never lets the page idle. Carry a **rendered-width assertion per skin**: a
   known string in the display face against a nonexistent family, failing if
   they match — with a discriminating string, since a 10px gap proves nothing.
   ***(Website Infrastructure, once `signal-check` merges — assigned 31 Aug.)***

   **Phase one shipped without touching activity chrome** — both activities
   diffed at 0 pixels. Phases two and three will need a baseline regeneration.

## 9. Open decisions — waiting on Maciej

**Nothing.** Old codes are **re-minted, not deleted** — decided 31 Aug, once the
cascade was measured. §8 carries the job; nothing is lost.

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
| **A phone joining a game gets a light border down both edges** | `globals.css:1710` sets `body { padding: 10px 6px }` under 640px, next to `.monitorFrame` — it is chrome for the CRT bezel. On a full-bleed page like `/live/join` there is no bezel, so the padding shows the page ground through it. Reads as a design choice rather than a fault, on the surface a child actually uses, and no contrast or pixel check looks at `body` padding | **open** · Website Infrastructure — `app/globals.css` is platform. Reported by the session that hit it rather than fixed, correctly: not its file |
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
| **Nothing verifies that the drawn things render** | No script mentions `hotspot`, `facsimile`, `phaseIcon` or `data-icon`; sixteen hotspots across twelve plate forms and seven glyphs were checked by hand, once, by one session. Rename `.plateRows` or change a `data-icon` and `doctor`, `e2e`, `tsc` and 131 unit tests all still pass. The page renders — it renders *wrong* | **§8's visual regression check** is the only cover this work will ever have |
| **A check that agrees with the bug it was written to catch** | Both fixes this session shipped with a check that passed for the wrong reason, each found only by deliberately breaking the thing it guarded. The import fixture passed against the *broken* importer until the fault was moved. The grant audit read a `pg_catalog` view that hides exactly the tables it was hunting — so it found nothing and reported clean, which is indistinguishable from finding nothing because nothing is wrong | **pattern, not a defect** — a guard is only proven by breaking its subject, and both were |
| **Signal Check's reveal is off by one, and it is live** | `revealFor` does `this.questions[position]` — a 0-based array indexed by a 1-based position — so the board shows the **next** question's answer, explanation and option labels. **The reported mis-scoring was never real** — that came from a harness reading canonical option order and skipping the per-player permutation. The header reads *QUESTION 2 OF 22* for the first question served. Found while staging a screenshot, isolated to one player and one answer, and verified against `session_questions` | **fixed and deployed** `9eb2071`. The constructor now refuses anything not renumbered, so the fault cannot return silently — it raises before a child joins |
| **The scrubber redacts the debug id, so source maps never resolve** | `UUID_ANYWHERE` in `lib/live/telemetry.ts` matches Sentry's own `debug_meta.images[].debug_id`, which is a UUID by design. Sentry discards it — *invalid debug identifier* — and with nothing joining the served bundle to the uploaded map, every frame arrives minified. **The maps upload correctly and are never applied.** The earlier instance of this ate `trace_id` and `public_key` and was found because it caused a 400; this one produces no error at all | **fixed** `87a737e` — the scrubber now names the fields that can carry what a child wrote and leaves the envelope alone. A production payload carries an intact debug id matching an uploaded map, so *invalid debug identifier* cannot recur. **A mapped frame is still unseen** |
| **Telemetry carries an identifier and nothing errors** | Sentry payloads are scrubbed at the client, so a regression in the scrubber sends codenames, guest nicknames or a game PIN to a third party while looking exactly like correct behaviour. Needs a guard that spells out its own patterns — one importing the scrubber's is a single gate, and empties when the scrubber breaks. `docs/identity-and-access.md` has the contract | **candidate** · Quiz Maker — before the first classroom session |
| `tracesSampleRate` is 1.0 | Measured: roughly **140 transactions** for a thirty-player session of three questions. Left untuned on purpose so the figure came from the harness rather than from anyone's estimate | **measured** — tune when a real session shape is known |
| **Throughput, not monthly volume, is the binding Realtime limit** | Measured 31 Aug against thirty real sockets: **96 msg/s peak**, 3.6 deliveries per player per question, 381 for a session. Supabase counts each delivery to each client, so one broadcast into a room of thirty is thirty events — which is why a monthly total was the wrong thing to worry about. Still does not error at the ceiling; it stops delivering | **measured**, and resolved by moving to Pro: 96 of 500 |
| **A session desyncs and one player's score diverges from the server's** | Client and server both hold a score. They can drift apart with nothing comparing them, and the child sees only their own number — so it reads as correct until the review screen, or until a teacher is asked why two agents with the same answers finished apart | **candidate** · Quiz Maker |
| Pruning a list by its length rather than its contents | This table was emptied over one session. Every removal was defensible alone — fixed, or recorded in `CLAUDE.md` — and nothing checked whether the section still said anything, because the number being watched was the line count | **fixed by rebuilding** — prune against what is still open, never against a budget |
