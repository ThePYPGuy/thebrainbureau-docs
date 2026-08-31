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
| **Doc Manager** | `STATUS.md`, `CLAUDE.md`, `docs/` — *not* `scripts/docs-sync.mjs`, which publishes them and is WI's | own, on `docs` | n/a — never in main's tree |

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
| Case File skin | `main` | none | Stages 1–2 done; 3 blocked on images |
| Prime Directive | `operation-prime-directive` | none | **identical to `main`** at `cac3f44` |
| Platform | `platform` in `../tbb-platform` | none | level with `main` |
| Docs | `docs` | `STATUS.md`, `CLAUDE.md` | own worktree; merges to `main` `--ff-only` |
| Signal Check | `signal-check` in `../tbb-quiz` | none | **complete on the branch** — 14 commits |

**Signal Check has started.** Transport interface, two implementations behind one
contract suite, the engine, the per-player shuffle, the telemetry scrubber, and
**Signal Check is complete on the branch** — 14 commits, 395 tests across 18
files and 45 e2e checks, all re-run here. The teacher session screen and the
accessibility switch were the last two, and both landed. Migration 26 is still
branch-only: not `main`, not production.

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

**A class code is open on production.** `OP-35HY` for Prime Directive, alongside
three older codes; two classes and three agents exist. Maciej is still the only
person who has *played* anything, so far as the repo can tell — but the door is
now open, and §10's rows about what would happen to a child stop being
hypothetical the moment one uses that code. Verified against production, not
assumed: `deployments` joined to `activities`.

**Only the main worktree can reach production.** `--prod` elsewhere fails with
*no linked Supabase project found*, which reads like a broken install and is a
boundary: you cannot write to production from a branch by accident. Run
production commands from the main worktree. (Why, and the override, in §5's
file.)

**What is proven is the database, not the running code.** `deploy:check --prod`
never asks Vercel which commit is serving. Confirmed *indirectly* 31 Aug:
`/api/bank-template` returns `text/csv` in production and that route exists only
in the merged code, so the deployment is current. The SHA itself is still
unasked — a route is evidence, not an identity.

**Production matches the repo** — `--prod` green after `b663ad3`: 25 migrations
applied, three activities and three training banks published, 73 skills, 20 tags. **Re-run after every
publish**: `?` is not `DRIFT` — it means unconfirmable, and the answer is
usually a re-import.

**Live:** the platform, three activities, the Case File skin and its CSS, the
evidence capability, and the `completion` gate. Prime Directive published at
`bb7f9b2` and imported to production; its art is on `main` and serving. The
question bank refactor is live at `b663ad3`.

**Signal Check is built and unshipped, and that includes Sentry.** `next.config.ts`
on `main` has no Sentry in it — checked, not assumed — so the production build
does not include the SDK and `SENTRY_AUTH_TOKEN` in Vercel is **inert until
`signal-check` merges**. Nothing is being monitored in production today, and the
readable-stack-trace check cannot run until after that merge and the deploy that
follows it.

The gate was **verified locally, not on production** *[verify — no session has
re-checked this since 25 Aug]* — proving it there means
creating a student agent in a live database. To confirm on the site, join a
real class code and check the ending stays absent until the last lock.

**Before any push** run
`git diff --stat origin/main..main -- supabase/migrations/ content/`. Neither
deploys itself and both fail silently when missed; no range so far has needed
one.

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
| Publish drift | `npm run deploy:check [-- --prod]` | built (`0760a88`); `--prod` fixed (`8839d38`) — exits rather than checking the wrong database |
| Line endings | `.gitattributes` | built (`f0563c2`) |
| Skin variety | `npm run skins` | built — reads the DB, not content files (§10) |
| Tests | `npm run test`, `npm run e2e` | vitest suite; 6 e2e scripts, incl. answer leak |
| Importer safety | `npm run test:reimport` | built |
| Targeted | `test:dashboard`, `:signup`, `:school`, `:entitlements`, `:curriculum` | built, 5 scripts |
| Bank checks | `test:columns`, `:insight`, `:images`, `:search`, `:edit-after-serve`, `:copy`, `check:alt`, `check:keys` | built, 8 scripts |

**Activity schema is locked at 0.4**; `activity-schema-v0.4.md` was never
written. **`npm run test` is green — 395/395, 18 files**, run on `signal-check`
31 Aug. Still no CI, so that number ages the moment anybody commits.

## 7. Recently completed

**A window, not a record** — oldest entries fall off, and git holds the rest.
This is the section that pays for the 250-line cap; §8 and §10 do not. Hashes
from `git log`; descriptions are each session's account of its own work.

- `b663ad3` — **the question bank refactor is live.** `training_sims` split into banks, questions, keys, sessions, snapshots, runs and responses, with mode config on the session and **none on the bank**. Three question types, CSV import, images with an alt-text check, per-year-group difficulty from raw serves, the insight view, search, question editing and *Take a copy*. 25 migrations and the storage bucket applied to production; `deploy:check --prod` clean before and after the re-import. **The new code is serving** — `/api/bank-template` answers `text/csv`, a route only it has. The backfill gave 4 banks, 66 questions and 66 keys, and turned one historical run into a session with 10 snapshot rows; the slugless teacher draft came across as `draft/private`, which is the row a title-join would have lost. *(WI; every check re-run by Doc Manager.)*
- `2b6df67`, `4983d89` — **the two authoring gaps closed**. `npm run skins` counts content files rather than rows, so it answers about what you are writing instead of what you have imported. And **a task can price its own hint**: `config.hintPenalty` overrides the activity's, falling back when absent, so Prime Directive's factors-and-primes hint need no longer cost the same as its subtraction hint. Every existing task keeps the old number. *(WI.)*
- `f7a33d6` — **`npm run import:one -- <slug|path>`**, so one drifted file can go to production without rewriting the other six; the `--prod --yes` passthrough is preserved. *(WI.)*
- `be460dc`, `aec8557` — **`npm run check:tokens`**, which fails on a hex colour or `font-family` written outside a token block: 43 existing literals grandfathered by name so the check lands green, and the list only shrinks. Four black slabs reached a light skin this week and each was one of these. **`npm run import` walks the directory** instead of naming every file, so a new activity no longer conflicts in `package.json` on every merge. *(WI.)*

## 8. Next up

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
   It sat unowned from the day it was filed, and an item with no name against it
   is a note rather than a plan. That is most of why it was deferrable.

## 9. Open decisions — waiting on Maciej

- **Merge `signal-check` and deploy?** Nothing waits on an agent: both pre-merge
  screens landed, 395 tests and 45 e2e checks pass, `doctor` is green. Bigger
  than the last merge in two ways — migration 26 carries `submit_live_answer`,
  which the atomic write depends on, and merging **turns Sentry on for the whole
  app**, not only this mode. Order: migrations first, then push, then read the
  Vercel build log, then confirm one stack trace.

## 10. Known silent failures

Open items. Standing traps are in `CLAUDE.md`; prune a row only when it is
closed there or here, and **never to hit a line count** — that emptied this
table once. **Documentation does not fire at 11pm.**

| Failure | Symptom | Owner |
|---|---|---|
| Short numeric answers are not leak-checked, and cannot be | `test-answer-leak.ts` derives forbidden values from each activity's own `answer` fields, `_`-prefixed subtrees and `completion` — but with two floors that **are** its coverage. `PROSE_FLOOR` 12 chars, below which a secret string is usually an identifier: `_evidenceDesign.column` is "colour", public by design, and forbidding it would fail on correct content. `DIGITS_FLOOR` 3, below which a figure cannot be told from any other on the page — Lock 06's answer is 2 and appears inside its own exhibit, correctly, because it is the sum. Lock 07 is covered only because its `prefix` makes the typed value `C-09` | **stated, not fixable** — written out in the script so it is never cited for more than it does |
| A check that resolves a tool by bare name | It reports the launching shell's `PATH` as the machine's state. `uv` is installed and `doctor` passes interactively and fails under `bash -c`, because `~/.local/bin` comes from `~/.profile`. Reported as *not installed*, which is a different problem with a different fix | **fixed** `84138c5` — it now names the path it found and says which of the two it means |
| A check that fires on every call is one nobody reads | The aspect-ratio check first compared reduced fractions and flagged a genuine 4:3 response of 1200×896 — 75:56, 0.45% out. Decimals within 2% now, tested against six cases including that one. Same lesson `doctor` already carries about its anchored `/images/` pattern | **fixed** in the art skill |
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
| **Telemetry carries an identifier and nothing errors** | Sentry payloads are scrubbed at the client, so a regression in the scrubber sends codenames, guest nicknames or a game PIN to a third party while looking exactly like correct behaviour. Needs a guard that spells out its own patterns — one importing the scrubber's is a single gate, and empties when the scrubber breaks. `docs/identity-and-access.md` has the contract | **candidate** · Quiz Maker — before the first classroom session |
| `tracesSampleRate` is 1.0 | Measured: roughly **140 transactions** for a thirty-player session of three questions. Left untuned on purpose so the figure came from the harness rather than from anyone's estimate | **measured** — tune when a real session shape is known |
| **Throughput, not monthly volume, is the binding Realtime limit** | Measured 31 Aug against thirty real sockets: **96 msg/s peak**, 3.6 deliveries per player per question, 381 for a session. Supabase counts each delivery to each client, so one broadcast into a room of thirty is thirty events — which is why a monthly total was the wrong thing to worry about. Still does not error at the ceiling; it stops delivering | **measured**, and resolved by moving to Pro: 96 of 500 |
| **A session desyncs and one player's score diverges from the server's** | Client and server both hold a score. They can drift apart with nothing comparing them, and the child sees only their own number — so it reads as correct until the review screen, or until a teacher is asked why two agents with the same answers finished apart | **candidate** · Quiz Maker |
| Pruning a list by its length rather than its contents | This table was emptied over one session. Every removal was defensible alone — fixed, or recorded in `CLAUDE.md` — and nothing checked whether the section still said anything, because the number being watched was the line count | **fixed by rebuilding** — prune against what is still open, never against a budget |
