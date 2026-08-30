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

**Only the main worktree can reach production.** `supabase/.temp/project-ref`
is gitignored, so it exists in `~/thebrainbureau` and nowhere else, and
`--prod` from a feature worktree fails with *no linked Supabase project found*.
That is a good property — you cannot write to production from a branch by
accident — but the error reads like a broken link rather than a deliberate
boundary. Run production commands from the main worktree.

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

## 8. Next up

1. **Maciej plays the finished version.** Everything Prime Directive needs is
   on production — schema, icons, tags, evidence, answer keys — and only its
   `draft` flag stands between it and a class. Op Builder has played it
   knowing every answer, which is how a missing Suspect Log survived a full
   pass. This is the last look anyone gets before children do.
2. **A stuck phase is unrecoverable for a child** — the one to fix first.
   Zero Hour lock 1 read *correct* while its phase never completed, so lock 2
   stayed locked. `settleCompletion` fires only on the transition to correct,
   so re-answering cannot recover it: a `task_progress` row without its
   `phase_progress` row is a dead end with no way out from the UI. Cause
   unknown — and **not** the "8 of 7 done" on the hub, which turned out to be a
   too-broad `UPDATE` in a local dev database, since corrected. That leaves
   this instance with no explanation at all. *(Website Infrastructure.)*
3. **No answer-leak guard covers Prime Directive** — the seven `e2e` scripts
   pass, but their leak checks cover Zero Hour and Global Intel Cards only.
   This activity's served state was verified by hand, and it is the one that
   already shipped an answer in a caption. *(WI.)*
4. **`Mission.tsx` hardcodes one activity's fiction for all**   shows "⚠ ZERO HOUR" and completes on "VAULT SECURE". A Case File
   short-circuit inherits the Value Vault. *(Website Infrastructure.)*
5. **Replace `--fd-scale` with explicit sizes** — a single scalar cannot describe a proportional face; Archivo Narrow measured 0.786–1.002 across six strings. Seventeen declarations shared with Field Terminal, so platform. *(WI.)*
   it and draw nothing. **Global Intel Cards is live and declares `"$"`.**
   *(Website Infrastructure.)*
6. Visual regression check — screenshot each activity, fail on change.
   **Clear intervals and cancel animations before capture** or the HUD timer
   never lets the page idle (Op Builder proved this out). Carry a
   **rendered-width assertion per skin**: a known string in the display face
   against a nonexistent family, failing if they match.
7. Lint rule on hex colours and `font-family` outside token blocks; make `"import"` a glob.

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

Open items. Standing traps are in `CLAUDE.md`; prune a row only when it is
closed there or here, and **never to hit a line count** — that emptied this
table once. **Documentation does not fire at 11pm.**

| Failure | Symptom | Owner |
|---|---|---|
| A phase that completes its task but not itself | The task reads *correct*, the next lock stays shut, and `settleCompletion` fires only on the transition — so re-answering cannot recover it. A child is simply stuck, with nothing to report | **§8.1** · Website Infrastructure |
| A failed import leaves rows it created | Positions are restored and creations are not — no transaction, because the Supabase client cannot open one; it would take a Postgres function. Bites only if the file is reverted after a failed import | **stated, unfixed** · Website Infrastructure |
| A test that does not reproduce the reported bug | The first recovery test poisoned the *last* phase, so the failure landed after all seven had been renumbered: it caught the ordering corruption and stranded nothing, and would have passed against the broken importer for the wrong reason. Moving the fault to the first phase reproduced it exactly | prove the test fails against the original bug, not a neighbour |
| Leak guards that cover only the activities they were written for | `e2e` passes while an untested activity ships its whole deduction. Prime Directive's served state is checked by hand, and it is the one that already shipped an answer in a caption | **§8.3** · Website Infrastructure |
| One activity's fiction hardcoded for all | Every activity shows "⚠ ZERO HOUR" and completes on "VAULT SECURE". A workshop short-circuit case ends by securing the Value Vault, and it renders perfectly | **§8.4** · Website Infrastructure |
| A single scalar describing a proportional face | `--fd-scale` was exact for two monospace faces and has no single value for Archivo Narrow — 0.786 to 1.002 across six strings. Wrong sizes render, they do not error | **§8.5** · Website Infrastructure |
| `npm run skins` reads the database, not the content files | Reports a stale archetype count, so the variety prompt advises on a world that may not be the repo | **unfixed** · Website Infrastructure |
| New tables need an explicit `service_role` grant | Reads fail as a permission error that reads like a missing row | **unfixed** · Website Infrastructure |
| Pruning a list by its length rather than its contents | This table was emptied over one session. Every removal was defensible alone — fixed, or recorded in `CLAUDE.md` — and nothing checked whether the section still said anything, because the number being watched was the line count | **fixed by rebuilding** — prune against what is still open, never against a budget |
