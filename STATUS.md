# Brain Bureau — STATUS

The coordination layer between parallel Claude Code sessions and the Claude
chat conversations used for project management. It records what is moving
*now*; permanent traps and conventions are in `CLAUDE.md`.

**Not a record of environment or deployment state** — that rots silently. Every
claim below names the command that produces it. **[verify]** means nobody has
checked; if this file and the repo disagree, **the repo wins.**

**Public**, mirrored to
[thebrainbureau-docs](https://github.com/ThePYPGuy/thebrainbureau-docs) by
`npm run docs:sync` — a mirror, never a second source of truth.

**Provenance.** §2–6 rest on commands run 2026-08-25: `doctor`, `deploy:check`
(local and `--prod`), `git log`/`status`/`worktree list`, `package.json` read
directly. Stage progress, §7's measurements and the key rotation come from the
sessions' own reports — attribution, not verification.

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

Getting there took two steps. `8839d38` fixed the check, which had printed
"checking PRODUCTION" while reading content from localhost. The first honest
run then returned five `?` — the rows predated content hashes, so nothing could
confirm them. A re-import against production stamped the hashes; the second run
came back clean. Content was byte-identical to `origin/main` beforehand, so
this changed fingerprints rather than content, and the in-place importer left
student progress intact.

Re-run it after every publish. `?` is not `DRIFT`: it means unconfirmable, and
the answer is usually a re-import.

**Deployed 2026-08-25.** `d446fb4..eb46bfc`, 13 commits — the Case File skin,
the `deploy:check` fix, `.gitattributes` and the documentation work. The Stage 2
hold was released early because the skin's regressions were verified in a
browser first.

**Neither manual step was needed**, checked rather than assumed: the range
touched no `supabase/migrations/` and no `content/`. Before any future push,
run `git diff --stat origin/main..main -- supabase/migrations/ content/` — both
of those fail silently when missed.

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

- **13 commits deployed** (`d446fb4..eb46bfc`) — first push since the Case File work began; no migrations or content in the range, so nothing manual followed.
- **Zero Hour checked in a browser** — both `31db28f` regressions confirmed fixed: the monitor frame holds 686px at 720/900/1100px windows, and task prose keeps the skin's face.
- **Production re-imported and verified clean** — first confirmed match between the deployed database and the repo.
- `b0ead55`–`5eff8d6` — `STATUS.md` into the repo, the allowlisted public docs mirror, and machine-specific values split into `docs/local/`.
- `f0563c2` — `.gitattributes`; line endings normalised on commit. *(WI)*
- `8839d38` — `deploy:check --prod` no longer queries localhost and calls it production. *(WI)*
- Supabase `service_role` key rotated after transcript exposure. *(WI, reported)*
- `31db28f` — skin font tokens namespaced; monitor stretch fixed. *(WI)*
- `735c6bb` — Case File skin, Stage 1 of 5: type tokenised, two contrast failures fixed.
- `b53ddf2` — Prime Directive: 7 phases, 7 answer keys, 6 hints. On its branch.

## 8. Next up

1. **Fix the importers' target resolution.** `scripts/import-activity.ts` and
   `import-training.ts` still read `SUPABASE_URL ?? localhost` — the exact
   pattern `deploy:check` was repaired for in `8839d38`. `npm run import` with
   no variables set writes to the local database and says nothing.
   *(Website Infrastructure.)*
2. **Settle whether VT323 actually loads** **[verify]**. Skin faces come from a
   network `@import` (`app/globals.css:7`); Bureau faces use `next/font/google`,
   which self-hosts. A failed request falls back to bare `monospace` silently,
   and sizes tuned to VT323's narrowness then render oversized. Reported by eye.
   *(Website Infrastructure.)*
3. Case File Stages 2–5 — dossier chrome, evidence capability (the one that
   matters), persistent panel, two correctness fixes.
4. **Look at each activity in a browser before the next push.** Measuring
   computed styles missed the bezel bug entirely; measuring *rendered geometry*
   across window heights catches it, and a person catches what neither does.
5. **Merge `main` into `operation-prime-directive`** before judging how Prime
   Directive looks — it has no skin yet (§3).
6. **Tag Prime Directive's curriculum skills** on merge (~10 min). Two gaps
   confirmed against `content/curriculum/`: no skill for *factors, multiples
   and primes* (nearest is a Y3 unit-fractions entry) and no *order of
   operations* at all. Both are UK Y6 blocks it builds locks on.
7. Visual regression check — screenshot each activity, fail on change.
8. Lint rule on hex colours and `font-family` outside a token block; make
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
- **Centre the monitor rather than top-align it?** `align-items: flex-start`
  correctly stopped the stretch, but a tall window now leaves ~200px dead below
  the monitor; `center` would stop it too and split the surplus. Not a defect —
  a look. Operation Builder owns the skin.
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
| Skin fonts load by network `@import`, Bureau fonts by `next/font` | A failed request falls back to bare `monospace`; looks plausible in a terminal skin, and sizes tuned to VT323 render oversized | **[verify] §8.2** · Website Infrastructure |
| `getComputedStyle().fontFamily` reports the *specified* family | Says `VT323` whether or not the face ever loaded — use `document.fonts.check()` | none — cost this session a wrong all-clear |
| Importers default to localhost when `SUPABASE_URL` is unset | `npm run import` writes to the laptop while appearing to publish | **§8.1** · Website Infrastructure |
| `npm run skins` reads the DB, not content files | Reports a stale count | Website Infrastructure |
| A branch lacking the skin its activity needs | Renders unstyled; reads as a CSS bug | candidate `doctor` check |
| Shared CSS changed for one skin | Another activity's look shifts, unnoticed | §8.6 |
| Fixed-height child in a stretching flex parent | Surplus renders as chrome, invisible at small heights | §8.6 |
| Evidence images not under version control | Not in `public/images/operations/<slug>/` | §9 |
| A doc referencing a file that does not exist | Reader hunts for it, or trusts a spec never written | Doc Manager — candidate link check |
