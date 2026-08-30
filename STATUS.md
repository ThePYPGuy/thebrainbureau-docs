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
- `b1e74ca` — **the seven glyphs, and the last claim line.** Both capability lists complete, so the publish gate is clear. Field Terminal verified two ways: no icon element renders, and one injected into a row computes `display: none` and moves it 0px. `interview` is a statement bubble, not the mockup's padlock — every locked row already shows a padlock, so it would have said "locked" on the one lock that is open. *(Op Builder.)*
- `66d6cdf`, `b46df7b` — **the icon field and the certificate print fix.** A named set in `lib/phase-icons.ts`, with the gate proved by rejection before being trusted to accept; absent renders no element at all, and the base rule is `display: none` so a skin must opt in. Certificate isolation re-keyed on `body:has(.certificate)` rather than the CRT chrome: Case File went 4 pages and 19 images to 1 and 0, Zero Hour still prints alone. Migration `20260830000015` applied to production. *(WI.)*
- `1a425ea`, `b832987`, `3e4638e` — **the Case File drawn.** Exhibits are CSS facsimiles in real text, twelve plate kinds across sixteen hotspots, and the on-page transcript came off *after* them. Index redrawn: ruled folder, ACTIVE CASE stamp that turns CASE CLOSED, progress bar counted in CSS, memo on its own sheet with a paperclip, CLEARED stamps. Five contrast failures found by measuring things that looked finished. Played through, all seven locks, every figure read off a drawn exhibit. *(Op Builder.)*
- `108652c`, `411e69f` — **Prime Directive unblocked.** Dossier render slot, briefing open on load, `prefix` drawn on both input paths, image zoom, `/terminal/print` with `app/print.css`, and `stripNotes` extended to `orders` at read time with e2e assertions beside the completion guard. *(WI.)*
- `f1cb2ce` — **Case File Stage 2.** Three defects Stage 1's measuring could not see, all found by looking: an unscoped `h2.caret::after` putting a terminal cursor on a paper dossier; `background: #000` literals in the shared `input`/`button` rules, which only look wrong on a light surface; `--accent` at 3.54:1 on manila, now 4.64:1. Callout constrained. *(Op Builder.)*
- `48a23b3`, `914d97b` — Field Terminal measured at 72ch (`ch` is honest there — Share Tech Mono's `0` and average glyph both 8.64px), callouts constrained from 101; `CI` exempts nothing. Both **live** — §10. *(WI)*
- `f04c924` — both viewports centred; Case File given a measure and a monochrome padlock. Settled by measurement at 1000px and 660px; the clipping objection does not apply because `.crtViewport` uses `min-height`. Both halves since confirmed by eye — centring and padlock correct. *(Op Builder.)*
## 8. Next up

1. **A stuck phase is unrecoverable for a child** — the one to fix first.
   Zero Hour lock 1 read *correct* while its phase never completed, so lock 2
   stayed locked. `settleCompletion` fires only on the transition to correct,
   so re-answering cannot recover it: a `task_progress` row without its
   `phase_progress` row is a dead end with no way out from the UI. Cause
   unknown — and **not** the "8 of 7 done" on the hub, which turned out to be a
   too-broad `UPDATE` in a local dev database, since corrected. That leaves
   this instance with no explanation at all. *(Website Infrastructure.)*
2. **The importer validates after it starts writing — fix before the next
   production import.** `import-activity.ts:135` bumps every phase position by
   +1000 to dodge the unique constraint; the icon check throws at :145, in the
   loop after. A mistyped icon name aborts the import and leaves the phases at
   1001–1007, which `Mission.tsx` pads to "1001". The activity stays corrupt
   until a successful import repairs it. **Publishing Prime Directive requires
   a production import**, so this is a precondition, not a nicety. Validate the
   whole file before the first write. *(Website Infrastructure.)*
3. **No answer-leak guard covers Prime Directive** — the seven `e2e` scripts
   pass, but their leak checks cover Zero Hour and Global Intel Cards only.
   This activity's served state was verified by hand, and it is the one that
   already shipped an answer in a caption. *(WI.)*
4. **`Mission.tsx` hardcodes one activity's fiction for all**   shows "⚠ ZERO HOUR" and completes on "VAULT SECURE". A Case File
   short-circuit inherits the Value Vault. *(Website Infrastructure.)*
5. **Three Y6 skills missing from the crosswalk** — *factors, multiples and
   primes* (Lock 04), *square numbers* (Lock 05), *order of operations*
   (Lock 06). Three locks stay untagged until they exist, and a near-miss tag
   is worse than none: it makes the Operation answer a search for something it
   does not teach. A curriculum decision. *(Maciej.)*
6. **Replace `--fd-scale` with explicit sizes** — a single scalar cannot describe a proportional face; Archivo Narrow measured 0.786–1.002 across six strings. Seventeen declarations shared with Field Terminal, so platform. *(WI.)*
   it and draw nothing. **Global Intel Cards is live and declares `"$"`.**
   *(Website Infrastructure.)*
7. **`.completeBox` renders black on manila** — the same unscoped-literal leak
   as Stage 2's input and button. *(Op Builder.)*
8. Visual regression check — screenshot each activity, fail on change.
   **Clear intervals and cancel animations before capture** or the HUD timer
   never lets the page idle (Op Builder proved this out). Carry a
   **rendered-width assertion per skin**: a known string in the display face
   against a nonexistent family, failing if they match.
9. Lint rule on hex colours and `font-family` outside token blocks; make `"import"` a glob.

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
| `.completeBox` unscoped black on a light skin | Fourth instance of a colour literal that only looks wrong on manila | **§8.4** · Op Builder |
