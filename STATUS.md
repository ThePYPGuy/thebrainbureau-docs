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
| Prime Directive | `operation-prime-directive` | none | pushed `77f1e83`; 2 behind again |
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

**Production matches the repo** — `--prod` green on all four sections, most
recently after `3e30eac`. **Re-run after every publish**: `?` is not `DRIFT`,
it means unconfirmable, and the answer is usually a re-import.

**Live:** the platform, two Field Terminal activities, the Case File skin, the
evidence capability, and the `completion` gate. **Not live:** Prime Directive —
content and images sit on its branch, so its art 404s in production, correctly,
and the Case File CSS styles nothing yet.

The gate was **verified locally, not on production** — proving it there means
creating a student agent in a live database. To confirm on the site, join a
real class code and check the ending stays absent until the last lock.

**Before any push** run
`git diff --stat origin/main..main -- supabase/migrations/ content/`. Neither
deploys itself and both fail silently when missed; no range so far has needed
one.

## 5. Environment

`npm run doctor` — 2026-08-25: **no failures, 2 warnings** (`uv` not on PATH,
`tools/google-image-gen` absent). Both expected — that pipeline is not in the repo.

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

Newest first. Hashes and order from `git log`; descriptions are each session's
own account of its own work, not re-measured here.

- **Doc Manager moved to its own worktree** on branch `docs` — the last shared-tree collision risk closed structurally rather than by care.
- **Prime Directive played end to end** as a fresh agent — the first time anyone has solved all seven locks rather than inspected them. All keys accepted, wrong answers refused with their own feedback, Intel 150/75/25 as designed (1,375 total, promoted), hints on 01–06 and none on 07, `completion` `{}` at 0/7 and 6/7 and released at 7/7, certificate unlocked. Two defects no check could see (§10) — but knowing the answers hid a third: the activity is unplayable. *(Op Builder.)*
- `14a726f`, `4d268bd`, merged in `3e30eac` — **`completion` released only once every phase is done**, with a check proved by reverting the gate: 4 assertions go red. Built against a fixture whose debrief names its own answer, since asserting on Zero Hour would pass for the reason the bug hid. Also checks it releases *whole* (`unlocksCertificate` rides along) and *per agent*. *(WI.)*
- `9405d9c` — **Stage 3 content.** Seven phases onto `config.evidence` + `_evidenceDesign`; `column` went private with them — it reads as a routing hint but names the Suspect Log column each lock filters on, and five of seven blocks held it. Seven images copied by name, byte-identical, no `_superseded/` shadows. *(Op Builder.)*
- `01182fd` — **Case File is designed** — drawn enough to ship, not finished. The test's expectation moved rather than the flag, so the next archetype meets the question rather than the answer. Five remain undrawn. *(WI.)*
- `b6f997b` — **the evidence boundary.** `config.evidence` is public (image, alt, caption); everything meaning-bearing goes behind `_`, `column` included — five of seven blocks are `{image, column}`, so that is most of them. One tested strip function replaces three copies of the rule, now applied on the way in *and* out. **[unpushed]** *(WI.)*
- `f1cb2ce` — **Case File Stage 2.** Three defects Stage 1's measuring could not see, all found by looking: an unscoped `h2.caret::after` putting a terminal cursor on a paper dossier; `background: #000` literals in the shared `input`/`button` rules, which only look wrong on a light surface; `--accent` at 3.54:1 on manila, now 4.64:1. Callout constrained. *(Op Builder.)*
- `48a23b3`, `914d97b` — Field Terminal measured at 72ch (`ch` is honest there — Share Tech Mono's `0` and average glyph both 8.64px), callouts constrained from 101; `CI` exempts nothing. Both **live** — §10. *(WI)*
- `f04c924` — both viewports centred; Case File given a measure and a monochrome padlock. Settled by measurement at 1000px and 660px; the clipping objection does not apply because `.crtViewport` uses `min-height`. Both halves since confirmed by eye — centring and padlock correct. *(Op Builder.)*
- `a50672b` — **the skin faces were never loading**; the terminal had drawn in bare monospace, ~⅓ oversized, since sizes are tuned to VT323's narrowness. All four now self-host via `next/font`. `--fd-scale` re-verified at 0.6671 against rendered widths — the 0.667 guess held to four decimals, and could not have been checked before. Method in `CLAUDE.md`. *(WI)*
- `fc6cffb` — importers refuse to write blind: no localhost default, and a confirmation before any remote write. *(WI)*
- **13 commits deployed** (`d446fb4..eb46bfc`) — first push since the Case File work began; no migrations or content in the range, so nothing manual followed.
- **Production re-imported and verified clean** — first confirmed match between the deployed database and the repo.
- `b0ead55`–`5eff8d6` — `STATUS.md` into the repo, the allowlisted public docs mirror, and machine-specific values split into `docs/local/`.
- `735c6bb` — Case File skin, Stage 1 of 5: type tokenised, two contrast failures fixed.
- `b53ddf2` — Prime Directive: 7 phases, 7 answer keys, 6 hints. On its branch.

## 8. Next up

1. **Build the Case Dossier — Prime Directive cannot be played without it.**
   The mechanic is elimination (`10-7-5-4-3-2-1`) and the ten suspects reach
   the student nowhere. **Settled by Maciej: on screen *and* a downloadable
   PDF.** The briefing is the first thing seen on load — typed on a page, not
   behind a disclosure — with the suspect list and portraits beside it.
   Nine portraits are still outside version control; only `cogsworth.png` is
   in. A suspect table is new public content: the columns are the ones
   `_evidenceDesign` names, and it must carry attributes without carrying the
   eliminations. **Split: the print route and base print stylesheet are
   platform (WI); the dossier's content and its `[data-skin="case-file"]`
   styling are Op Builder's.**
2. **Three Y6 skills missing from the crosswalk** — *factors, multiples and
   primes* (Lock 04), *square numbers* (Lock 05), *order of operations*
   (Lock 06). Three locks stay untagged until they exist, and a near-miss tag
   is worse than none: it makes the Operation answer a search for something it
   does not teach. A curriculum decision. *(Maciej.)*
3. **`prefix` is typed but never rendered** — `Tasks.tsx:316,470` destructure
   it and draw nothing. **Global Intel Cards is live and declares `"$"`.**
   *(Website Infrastructure.)*
4. **`.completeBox` renders black on manila** — the same unscoped-literal leak
   as Stage 2's input and button. *(Op Builder.)*
5. Visual regression check — screenshot each activity, fail on change.
   **Clear intervals and cancel animations before capture** or the HUD timer
   never lets the page idle (Op Builder proved this out). Carry a
   **rendered-width assertion per skin**: a known string in the display face
   against a nonexistent family, failing if they match.
6. Lint rule on hex colours and `font-family` outside token blocks; make `"import"` a glob.

## 9. Open decisions — waiting on Maciej

- **The Drive "Accounts" doc holds live credentials in plain text** — a
  database password and a mail-provider key; one `service_role` key has
  already been rotated this week after a transcript leak.
- **Duplicate style templates** in Maciej's image folder; `content/styles/`
  is authoritative. Unresolved across three revisions.
- **Nine suspect portraits stay outside version control**, awaiting a Suspect
  Log panel that does not exist — and the design leans on a *printable* one for
  paper cross-referencing. The seven referenced are in `9405d9c`.

## 10. Known silent failures

Open items, plus fixes recent enough to still be worth seeing — a fix names
the check that catches the thing. Standing traps are in `CLAUDE.md`. Prune a
row once its fix is old news. **Documentation does not fire at 11pm** — a
session hit the documented apostrophe trap two commits after documenting it.

| Failure | Symptom | Owner |
|---|---|---|
| A session commits to `main` mid-task | Another session's uncommitted files land in a commit that does not describe them | Website Infrastructure — candidate `doctor` warning on a dirty tree |
| Renormalising rewrites the working tree | Uncommitted edits silently revert; no error, no conflict | as above; cost Doc Manager `STATUS.md` on 2026-08-25 |
| An exemption whose condition is the hazard | Third instance in one file: the non-TTY skip, then `process.env.CI`. Both named the situation with nobody watching and then waived the guard for it | **fixed** `914d97b` — `CI` exempts nothing; an automated write types `--yes` like anyone else |
| Pushing publishes every session's unpushed commits | Three times. `git push` sends the branch, so a commit on `main` ships whenever anyone else pushes. The check worked on the third: the pusher read the range first, saw whose it was, and judged it — which is the difference the row buys. It still captures the **pusher's** intent, never the **author's** | **§8.1** — a branch per session is the actual fix |
| A demo is not a build | The suspect list and image zoom existed in a demo and never in this repo — `git log --all -S "suspect"` finds no component in any commit. Nothing carried them across because nothing was asked to | if it is not in a commit, it does not exist |
| An activity verified end to end that cannot be played | Every lock accepted its key, Intel and hints behaved, the gate held — and the deduction is impossible, because the suspects are not rendered anywhere. A session that knows the answers cannot detect a missing affordance | **§8.1** — playthroughs must be by someone who does not know the answer |
| The briefing is a collapsed `<details>` | `Mission.tsx:219` puts the whole back story behind a summary line that reads as a heading. Prime Directive's briefing exists and is good; nobody sees it | **unfixed** · Website Infrastructure — shared chrome |
| Evidence images cannot be enlarged | Exhibits carry figures a child must read — a batch sheet, a power log — rendered at fixed width and `pixelated`, with no zoom | **unfixed** · Website Infrastructure |
| A public field carrying something secret | `config.evidence.caption` read "C-09 COGSWORTH", publishing Lock 07's answer at 0 of 7 phases. The boundary held, the gate worked, `e2e` passed — the field is *meant* to be public, so no structural check can see this. **Found by playing** | **fixed** — read what public fields say, not just where they sit |
| `prefix` typed but never rendered | `Tasks.tsx` destructures it and draws nothing, so a Suspect Log of C-01…C-10 offers a numeric box that refuses `C-09`. Live: Global Intel Cards declares `"$"` | **§8.3** · Website Infrastructure |
| `.completeBox` unscoped black on a light skin | Fourth instance of a colour literal that only looks wrong on manila | **§8.4** · Op Builder |
| `npm run skins` reads the DB, not content files | Reports a stale count | Website Infrastructure |
