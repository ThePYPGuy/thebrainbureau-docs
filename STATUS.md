# Brain Bureau — STATUS

The coordination layer between parallel Claude Code sessions and the Claude
chat conversations used for project management. It records what is moving
*now*; permanent traps and conventions are in `CLAUDE.md`.

**Not a record of environment or deployment state** — that rots silently. Every
claim names the command producing it; **[verify]** means nobody has checked. If
this file and the repo disagree, **the repo wins.**

**Public**, mirrored to
[thebrainbureau-docs](https://github.com/ThePYPGuy/thebrainbureau-docs) by
`npm run docs:sync` — a mirror, never a second source of truth.

**Provenance.** §2–6 rest on commands run 2026-08-25: `doctor`, `deploy:check`
(local and `--prod`), `git log`/`status`/`worktree list`, `package.json` read
directly. Stage progress and the key rotation come from the sessions’ own
reports — attribution, not verification.

§7’s font findings are the exception: measured in a running browser this session,
not reported. Rendered DOM width against a deliberately nonexistent family —
the only method that gave a true answer, and the reason §10 now says every
cheaper check is wrong.

## 1. Sessions

| Name | Owns | Worktree | Holding the tree? |
|---|---|---|---|
| **Operation Builder** | Prime Directive + the Case File skin | own, on `case-file` + PD | n/a |
| **Website Infrastructure** | Platform, engine, scripts, checks | own, on `platform` | n/a |
| **Doc Manager** | `STATUS.md`, `CLAUDE.md`, `docs/` | own, on `docs` | n/a — never in main's tree |

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
| Platform | `main` | none | measure and CI exemption landed |
| Docs | `docs` | `STATUS.md`, `CLAUDE.md` | own worktree; merges to `main` `--ff-only` |

## 3. Overlap risks — READ BEFORE ASSIGNING WORK

Live collisions only; standing hazards are in `CLAUDE.md`.

- **A branch merge is a snapshot, not a subscription.** `operation-prime-directive`
  is 15 behind. Run `git rev-list --left-right --count main...HEAD` before
  judging how anything on a branch looks — never the last merge date.
- **`package.json` `"import"` is a hardcoded list, not a glob** — every new
  activity conflicts on merge; resolution is always "keep both sides".
- **`bb49a62` duplicates `5fda3d3`** — verified; cherry-pick residue is inference. Likely to conflict on merge.

## 4. Publish state

Pushing to `main` deploys the app. **Migrations do not run themselves. Content
does not import itself.** Both fail silently.

**Live at https://thebrainbureau.app** — the apex 308-redirects to `www`, so a
bare 200-or-fail check reads a healthy site as down. Follow redirects, or check
the `www` host.

**No class has used the platform yet.** All three activities are published and
the site is up, but Maciej is the only person who has played anything. "Live"
means reachable, not in use — worth holding when reading §10, where several
rows describe what *would* happen to a child rather than what has.

**Only the main worktree can reach production.** `--prod` elsewhere fails with
*no linked Supabase project found*, which reads like a broken install and is a
boundary: you cannot write to production from a branch by accident. Run
production commands from the main worktree. (Why, and the override, in §5's
file.)

**Production matches the repo** — `--prod` green after `bb7f9b2`: 15 migrations
applied, three activities published, 73 skills, 20 tags. **Re-run after every
publish**: `?` is not `DRIFT` — it means unconfirmable, and the answer is
usually a re-import.

**Live:** the platform, three activities, the Case File skin and its CSS, the
evidence capability, and the `completion` gate. Prime Directive published at
`bb7f9b2` and imported to production; its art is on `main` and serving. Nothing
is built but unshipped.

This paragraph said the opposite for several hours after the fact — *not live,
art 404s, CSS styles nothing* — every clause true when written. The publish step
in `CLAUDE.md` exists because of it, and §10's row on notes recording an absence
is about the same failure in the activity file.

The gate was **verified locally, not on production** — proving it there means
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

**Activity schema is locked at 0.4**; `activity-schema-v0.4.md` was never
written. **`npm run test` is green** — 131/131, 8/8 files, after running red
from `735c6bb` to `01182fd` with no CI to say so.

## 7. Recently completed

**A window, not a record** — oldest entries fall off, and git holds the rest.
This is the section that pays for the 250-line cap; §8 and §10 do not. Hashes
from `git log`; descriptions are each session's account of its own work.

- `2b6df67`, `4983d89` — **the two authoring gaps closed**. `npm run skins` counts content files rather than rows, so it answers about what you are writing instead of what you have imported. And **a task can price its own hint**: `config.hintPenalty` overrides the activity's, falling back when absent, so Prime Directive's factors-and-primes hint need no longer cost the same as its subtraction hint. Every existing task keeps the old number. *(WI.)*
- `bb7f9b2` — **Prime Directive is published** and imported to production; every line of the import said REMOTE, and `deploy:check --prod` reports 15 migrations, three activities, 73 skills, 20 tags. **Not deployed to a class** — that step mints the code and is Maciej's. Three stale `_note` sentences went out with it; a fourth, claiming the printable Suspect Log was never made, survived because Doc Manager vouched for it. *(OB.)*
- `f7a33d6` — **`npm run import:one -- <slug|path>`**, so one drifted file can go to production without rewriting the other six; the `--prod --yes` passthrough is preserved. *(WI.)*
- `be460dc`, `aec8557` — **`npm run check:tokens`**, which fails on a hex colour or `font-family` written outside a token block: 43 existing literals grandfathered by name so the check lands green, and the list only shrinks. Four black slabs reached a light skin this week and each was one of these. **`npm run import` walks the directory** instead of naming every file, so a new activity no longer conflicts in `package.json` on every merge. *(WI.)*
- **`--fd-scale` replaced by explicit sizes** — the token is gone; only two comments naming it survive, both explaining why a scalar could not work. Case File states its display sizes outright rather than deriving them, so nineteen values that were each the smallest size that could not overflow are now each the size they should be. Training's six stay Field Terminal-only until that archetype has a skin. **[on `platform`, unmerged]** *(WI.)*
- **Production re-imported through the glob** — checked first that only Zero Hour drifted, then verified against the data: its fiction is back (`⚠ ZERO HOUR`, `VAULT SECURE`, `RESTORE CODE`) while Global Intel Cards and Prime Directive take the colourless defaults, which is what `c11d11e` was for. No phase stranded above 1000 on the first production run under the new importer. Every line said `REMOTE`. *(WI.)*
- `c11d11e` — **one activity's fiction stops being every activity's.** The timer's end-state labels and the keypad label are authored now, defaulting to words that fit any case rather than to Zero Hour's. `docs/activity-file.md` written alongside: there was no activity-file reference at all, so every field found this week — `icon`, `prefix`, `evidence`, `dossier`, and now three labels — was documented only in the code that read it. *(WI, doc by Doc Manager.)*
- `84138c5` — **two `doctor` checks that described the shell instead of the machine.** `uv` now reports *installed at `~/.local/bin/uv`, but not on this shell's `PATH`* — verified from a stripped shell — instead of *not on PATH*, which read as *not installed*. And the `tools/google-image-gen` warning is retired in favour of a check for something the art skill actually needs: the API key. *(WI.)*
- `03d6907`, merged `8f84348` — **`.claude/skills/bureau-art/`**, the repo's first `.claude/` tree: generating operation art *and* the half that has broken twice — read the slug from the activity file, absolute paths, `git add`, `doctor` as the finish line. It does not publish; the mirror allows `docs/**.md` only, verified by `docs:sync --dry-run`. *(Op Builder.)*
- `df82038` — **answer values in public fields are checked**, derived from each activity's own secret fields rather than a list, so it covers an activity not yet written. Two floors, stated in §10, that are its coverage. *(WI.)*

## 8. Next up

1. **The art skill quotes an error `doctor` no longer prints.**
   `.claude/skills/bureau-art/SKILL.md:40` says the report reads `uv not on
   PATH`, "which reads like *not installed* and is not the same thing" — the
   exact complaint `84138c5` fixed. `doctor.ts:128` now prints the version and
   path *and* says it is not on this shell's PATH. So the skill teaches a
   workaround for a confusion that no longer exists, in a file whose whole
   point is that the distinction matters. One line. *(Whoever owns the art
   skill — not Doc Manager's file.)*
2. Visual regression check — screenshot each activity, fail on change.
   **Clear intervals and cancel animations before capture** or the HUD timer
   never lets the page idle. Carry a **rendered-width assertion per skin**: a
   known string in the display face against a nonexistent family, failing if
   they match — with a discriminating string, since a 10px gap proves nothing.

## 9. Open decisions — waiting on Maciej

- **Deploy Prime Directive to a class?** Published and on production; the
  dashboard step that mints a class code is the only thing left, and it is the
  first time a child rather than Maciej reaches an Operation.
- **The Drive "Accounts" doc holds live credentials in plain text** — a
  database password and a mail-provider key; one `service_role` key has
  already been rotated this week after a transcript leak.

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
| A failed import leaves rows it created | Positions are restored and creations are not — no transaction, because the Supabase client cannot open one; it would take a Postgres function. Bites only if the file is reverted after a failed import | **stated, unfixed** · Website Infrastructure |
| A test that does not reproduce the reported bug | The first recovery test poisoned the *last* phase, so the failure landed after all seven had been renumbered: it caught the ordering corruption and stranded nothing, and would have passed against the broken importer for the wrong reason. Moving the fault to the first phase reproduced it exactly | prove the test fails against the original bug, not a neighbour |
| Leak guards that cover only the activities they were written for | `e2e` passes while an untested activity ships its whole deduction. Prime Directive's served state is checked by hand, and it is the one that already shipped an answer in a caption | **fixed** — `scripts/test-answer-leak.ts` reads the directory and derives each forbidden value from the file, so a new activity is covered by construction rather than by being remembered |
| One activity's fiction hardcoded for all | Every activity shows "⚠ ZERO HOUR" and completes on "VAULT SECURE". A workshop short-circuit case ends by securing the Value Vault, and it renders perfectly | **fixed** — `Mission.tsx` takes `expiredLabel` and `doneLabel` from the theme, defaulting to `⏱ TIME UP` and `COMPLETE`. The Zero Hour literals survive only in a comment recording why |
| `npm run skins` counted the database, not the content files | It reported an archetype count from rows, so the file you were authoring — not imported yet — did not count, and a row whose file was gone still did. Wrong in the one moment the tool is used | **fixed** `2b6df67` — it reads `content/activities` |
| New tables need an explicit `service_role` grant | Reads fail as a permission error that reads like a missing row | **unfixed** · Website Infrastructure |
| Pruning a list by its length rather than its contents | This table was emptied over one session. Every removal was defensible alone — fixed, or recorded in `CLAUDE.md` — and nothing checked whether the section still said anything, because the number being watched was the line count | **fixed by rebuilding** — prune against what is still open, never against a budget |
