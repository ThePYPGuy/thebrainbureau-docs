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

| Name | Owns | Currently |
|---|---|---|
| **Operation Builder** | Prime Directive + the Case File skin | Active — Stage 1 of 5 done, Stage 2 not started |
| **Website Infrastructure** | Platform, engine, scripts, checks | Active — shipped `8839d38`, `f0563c2` |
| **Doc Manager** | `STATUS.md`, `CLAUDE.md`, `docs/` | Active — touches no code |

**One session on `main` at a time** — collided four times on 2026-08-25, the
last losing work. Mechanics in `CLAUDE.md`.

## 2. Active work

| Stream | Folder | Branch | Files | Status |
|---|---|---|---|---|
| Case File skin | main worktree | `main` | none | Stage 1 of 5 done |
| Prime Directive | PD worktree | `operation-prime-directive` | none | pushed, in sync, **not merged** |
| Website Infrastructure | main worktree | `main` | none — committed | `--prod` fix landed |
| Doc Manager | main worktree | `main` | `STATUS.md` | this change |

## 3. Overlap risks — READ BEFORE ASSIGNING WORK

Live collisions only; standing hazards are in `CLAUDE.md`.

- **Prime Directive is a Case File activity and its branch lacks the Case File
  skin.** `main` holds four commits the branch does not, including `735c6bb`,
  which *is* the skin. It cannot render as designed until `main` merges in.
- **`npm run doctor` does not exist on `operation-prime-directive`** — it
  arrived in `d446fb4`, after that branch last took `main`. Verified:
  `grep -c doctor package.json` returns 0.
- **`package.json` `"import"` is a hardcoded list, not a glob.** Every new
  activity conflicts on merge; resolution is always "keep both sides". The
  `import:curriculum` conflict was already resolved by merge `46d7037`.
- **`bb49a62` duplicates `5fda3d3`** — same message on both. The duplicate is
  verified; cherry-pick residue is inference. Likely to conflict on merge.

## 4. Publish state

Pushing to `main` deploys the app. **Migrations do not run themselves. Content
does not import itself.** Both manual parts fail silently.

**Production is verified** — 2026-08-25, for the first time.
`deploy:check -- --prod` queried the real host and reported every section `ok`:
14 migrations applied, both activities published, three training banks, 70
skills and 14 curriculum tags. **This database matches the repo.**

`8839d38` fixed the check; the first honest run returned five `?` because the
rows predated content hashes; a re-import stamped them and the second run was
clean. **Re-run after every publish.** `?` is not `DRIFT` — it means
unconfirmable, and the answer is usually a re-import.

**Deployed twice on 2026-08-25.** `d446fb4..eb46bfc` carried the Case File
skin, the `deploy:check` fix and the docs work; the second push carried the
self-hosted faces and the importer guards. Re-verified after landing: public
pages 200, `/dashboard` 307, **zero** `fonts.googleapis.com` requests in the
served HTML, `--prod` green on all four sections.

**Neither manual step was needed in either range**, checked rather than
assumed — no `supabase/migrations/`, no `content/`. Before any push, run
`git diff --stat origin/main..main -- supabase/migrations/ content/`; both fail
silently when missed.

## 5. Environment

`npm run doctor` — 2026-08-25: **no failures, 2 warnings** (`uv` not on PATH,
`tools/google-image-gen` absent). Both expected — that pipeline is not in the repo.

Paths, worktrees, project refs and Maciej's folders are in
`docs/local/environment.md`; `git worktree list` and `doctor` outrank it.
`.env.local` points at **local** Supabase and takes the *local* secret key —
production keys belong in Vercel only. What `doctor` does *not* assert is in
`CLAUDE.md`.

## 6. What is actually built

Verified against `package.json` directly, not against a summary of it.

| Check | Command | State |
|---|---|---|
| Preflight | `npm run doctor` | built (`d446fb4`) |
| Publish drift, local | `npm run deploy:check` | built (`0760a88`) |
| Publish drift, prod | `deploy:check -- --prod` | **fixed** (`8839d38`) — exits rather than checking the wrong database |
| Line endings | `.gitattributes` | built (`f0563c2`) |
| Skin variety | `npm run skins` | built — reads the DB, not content files (§10) |
| Tests | `npm run test`, `npm run e2e` | vitest suite; 6 e2e scripts, incl. answer leak |
| Importer safety | `npm run test:reimport` | built |
| Targeted | `test:dashboard`, `:signup`, `:school`, `:entitlements`, `:curriculum` | built, 5 scripts |
| Docs mirror | `npm run docs:sync` | built — `--dry-run`, `--no-push` |

**Activity schema is locked at 0.4** — both shipped activities declare it;
earlier revisions said 0.3 and were wrong. `CLAUDE.md` pointed at
`activity-schema-v0.4.md`, which **has never been written**; it now points at
the README.

## 7. Recently completed

Newest first. Hashes and order from `git log`; descriptions are each session's
own account of its own work, not re-measured here.

- `a50672b` — **the skin faces were never loading**; the terminal had drawn in bare monospace, ~⅓ oversized, since sizes are tuned to VT323's narrowness. All four now self-host via `next/font`. `--fd-scale` re-verified at 0.6671 against rendered widths — the 0.667 guess held to four decimals, and could not have been checked before. Method in `CLAUDE.md`. *(WI)*
- `fc6cffb` — importers refuse to write blind: no localhost default, and a confirmation before any remote write. *(WI)*
- **13 commits deployed** (`d446fb4..eb46bfc`) — first push since the Case File work began; no migrations or content in the range, so nothing manual followed.
- **Zero Hour checked in a browser** — both `31db28f` regressions confirmed fixed: the monitor frame holds 686px at 720/900/1100px windows, and task prose keeps the skin's face.
- **Production re-imported and verified clean** — first confirmed match between the deployed database and the repo.
- `b0ead55`–`5eff8d6` — `STATUS.md` into the repo, the allowlisted public docs mirror, and machine-specific values split into `docs/local/`.
- `f0563c2` — `.gitattributes`; line endings normalised on commit. *(WI)*
- `8839d38` — `deploy:check --prod` no longer queries localhost and calls it production. *(WI)*
- `735c6bb` — Case File skin, Stage 1 of 5: type tokenised, two contrast failures fixed.
- `b53ddf2` — Prime Directive: 7 phases, 7 answer keys, 6 hints. On its branch.

## 8. Next up

1. **Narrow `confirmRemoteWrite`'s `CI` exemption** (§10) — one line; the guard
   is otherwise sound. *(Website Infrastructure.)*
2. **Centre both viewports** — settled on measurement, not taste: at 1000px
   `flex-start` gives 30/284, `center` gives 157/157; at 660px, where the
   monitor overflows, all options behave alike with the top reachable, because
   `.crtViewport` uses `min-height` and grows. The clipping objection does not
   apply. Op Builder takes both halves, having done the measuring.
3. **Zero Hour's measure runs to 87 characters** — flagged by Op Builder, owned
   by Website Infrastructure: Field Terminal, live, and a readability change for
   Y6 readers rather than the alignment question asked. Case File's equivalent
   is Op Builder's, in Stage 2.
4. Case File Stages 2–5 — dossier chrome, evidence capability (the one that
   matters), persistent panel, two correctness fixes.
5. **Look at each activity in a browser before the next push.** Computed styles
   missed the bezel bug; rendered geometry catches it; a person catches what
   neither does — the fonts were found by eye after three checks said fine.
6. **Merge `main` into `operation-prime-directive`** before judging how Prime
   Directive looks — it has no skin yet (§3).
7. **Tag Prime Directive's curriculum skills** on merge (~10 min). Two gaps
   confirmed against `content/curriculum/`: no *factors, multiples and primes*
   (nearest is a Y3 unit-fractions entry) and no *order of operations*. Both
   are UK Y6 blocks it builds locks on.
8. Visual regression check — screenshot each activity, fail on change.
   **Include a rendered-width assertion per skin**: measure a known string in
   the skin's display face against a deliberately nonexistent family and fail
   if they match. That is the only check that would have caught the fonts —
   see §10.
9. Lint rule on hex colours and `font-family` outside a token block; make
   `"import"` a glob rather than a hardcoded list.

## 9. Open decisions — waiting on Maciej

- **The Drive "Accounts" doc holds live credentials in plain text** — a
  database password and a mail-provider API key. A `service_role` key was
  already rotated once this week after landing in a transcript.
- **Duplicate style templates** in Maciej's image folder; `content/styles/` is
  authoritative. Unresolved across three revisions.
- **Evidence images outside version control** — 6 PNGs, confirmed by listing,
  beside a `_superseded/` folder of 7 older ones under near-identical names.
  Stage 3 copies the 6, not the directory: a superseded image renders
  perfectly, so nothing reports the mistake.
- **Case File is `designed: true` with Stage 2 of 5 to come**
  (`lib/skins.ts:46`). Does the flag mean "drawn enough to ship" or "finished"?
- **Merge `operation-prime-directive` now, or wait for evidence capability?**

## 10. Known silent failures

Open items only, each with an owner; failures already caught by a check, and
standing traps, are in `CLAUDE.md`. **Documentation does not fire at 11pm** — a
session hit the documented apostrophe trap two commits after documenting it.

| Failure | Symptom | Owner |
|---|---|---|
| A session commits to `main` mid-task | Another session's uncommitted files land in a commit that does not describe them | Website Infrastructure — candidate `doctor` warning on a dirty tree |
| Renormalising rewrites the working tree | Uncommitted edits silently revert; no error, no conflict | as above; cost Doc Manager `STATUS.md` on 2026-08-25 |
| A safeguard whose exemption covers the case it was written for | `confirmRemoteWrite` correctly refuses a non-TTY run, then returns early on `process.env.CI` — set automatically by every CI, which is exactly where no human reads the warning | **§8.1** · Website Infrastructure |
| **A webfont that never loads** | Text renders in fallback at the wrong width; in a monospace skin it looks plausible, and sizes tuned to a narrow face come out about a third oversized | **fixed `a50672b`** by self-hosting — but see the row below for why it went unseen for so long |
| **Every cheap way of checking a font is wrong** | `getComputedStyle().fontFamily` returns what CSS *asked for*, so it said `VT323` regardless. `document.fonts.check('16px "VT323"')` returned **`true`** for a family not in the registry at all — it answers "would this be used", assuming system availability. `canvas.measureText` reported every family, real or invented, as an identical width. | **no check** · Website Infrastructure — the only honest test is rendered DOM width against a bogus family; candidate for §8.6 |
| Importers default to localhost when `SUPABASE_URL` is unset | `npm run import` writes to the laptop while appearing to publish | **§8.1** · Website Infrastructure |
| `npm run skins` reads the DB, not content files | Reports a stale count | Website Infrastructure |
| A branch lacking the skin its activity needs | Renders unstyled; reads as a CSS bug | candidate `doctor` check |
| Shared CSS changed for one skin | Another activity's look shifts, unnoticed | §8.6 |
| Fixed-height child in a stretching flex parent | Surplus renders as chrome, invisible at small heights | §8.6 |
| Evidence images not under version control | Not in `public/images/operations/<slug>/` | §9 |
| A doc referencing a file that does not exist | Reader hunts for it, or trusts a spec never written | Doc Manager — candidate link check |
