# Brain Bureau — STATUS

The coordination layer between parallel Claude Code sessions and the Claude
chat conversations used for project management. It records what is moving
*now*; permanent traps and conventions are in `CLAUDE.md`.

**It is not a record of environment or deployment state** — that rots silently.
Every claim below names the command that produces it. **[verify]** means nobody
has checked, and **if this file and the repo disagree, the repo wins.**

**Public**, mirrored to
[thebrainbureau-docs](https://github.com/ThePYPGuy/thebrainbureau-docs) by
`npm run docs:sync` — a mirror, never a second source of truth.

**Provenance.** §2–6 rest on commands run 2026-08-25: `doctor`,
`deploy:check` (local and `--prod`), `git log`/`status`/`worktree list`, and
`package.json` read directly. Stage progress, every measurement in §7, and the
key rotation come from the sessions' own reports — attribution, not
verification.

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
| Doc Manager | main worktree | `main` | `STATUS.md`, `CLAUDE.md` | this change |

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
- **`bb49a62` duplicates `5fda3d3`** — same commit message on the branch and on
  `main`. The duplicate is verified; that it is cherry-pick residue is
  inference. Likely to conflict on merge.

## 4. Publish state

Pushing to `main` deploys the app. **Migrations do not run themselves. Content
does not import itself.** Both manual parts fail silently.

**`deploy:check -- --prod` is fixed** (`8839d38`). It used to print "checking
PRODUCTION", compare migrations against the real project, then read all content
from localhost and report a match — half the report true, which is what made it
convincing. Verified by running it: with no credentials it now **exits** rather
than checking something else. **Production content state is unverified**, but
honestly so rather than falsely green.

`npm run deploy:check` (local) — 2026-08-25, after the fix:

```
14 migrations · global-intel-cards, operation-zero-hour · syntax-vault,
figurative-frequencies, multiplication-firewall · 70 skills, 14 tags — matches
```

**Unpushed:** `main` is deliberately ahead of `origin/main` — Case File is
held until Stage 2 lands. No count here; every revision that gave one was stale
within the hour. Run `git rev-list --count origin/main..main`.

## 5. Environment

`npm run doctor` — 2026-08-25: **no failures, 2 warnings** (`uv` not on PATH,
`tools/google-image-gen` absent). Both expected; the image pipeline lives
outside the repo.

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
| Unit tests | `npm run test` | vitest |
| E2E / answer leak | `npm run e2e` | 6 scripts |
| Importer safety | `npm run test:reimport` | built |
| Targeted | `test:dashboard`, `:signup`, `:school`, `:entitlements`, `:curriculum` | built, 5 scripts |
| Docs mirror | `npm run docs:sync` | built — `--dry-run`, `--no-push` |

**Activity schema is locked at 0.4** — both shipped activities declare it;
earlier revisions said 0.3 and were wrong. `CLAUDE.md` pointed at
`activity-schema-v0.4.md`, which exists nowhere and **has never been written**;
the reference now goes to the README.

## 7. Recently completed

Newest first. Hashes and order from `git log`; descriptions are each session's
own account of its own work, not re-measured here.

- `5eff8d6`, `1050219` — `docs:sync` separates a clean tree from a published one, and reports links that will not resolve in the mirror.
- `3b3ff44` — machine-specific values split into `docs/local/`, held out of the mirror.
- `f0563c2` — `.gitattributes`; line endings normalised on commit. *(WI)*
- `8839d38` — `deploy:check --prod` no longer queries localhost and calls it production. *(WI)*
- `b0ead55` — `STATUS.md` into the repo, plus an allowlisted `docs:sync`.
- Supabase `service_role` key rotated after transcript exposure. *(WI, reported)*
- `31db28f` — skin font tokens namespaced; monitor stretch fixed. *(WI)*
- `735c6bb` — Case File skin, Stage 1 of 5: type tokenised, two contrast failures fixed.
- `d446fb4` — doctor. `1c06de1` — image styles into the repo. *(WI)*
- `b53ddf2` — Prime Directive: 7 phases, 7 answer keys, 6 hints. On its branch.

## 8. Next up

1. **Run `deploy:check -- --prod` with credentials** and re-import on drift.
   The tool is trustworthy now; the answer is still unknown.
2. Case File Stages 2–5 — dossier chrome, evidence capability (the one that
   matters), persistent panel, two correctness fixes.
3. **Push `main` once Stage 2 lands** — look at Zero Hour in a browser first;
   `735c6bb` was verified by measuring computed styles, not by eye.
4. **Merge `main` into `operation-prime-directive`** before judging how Prime
   Directive looks — it has no skin yet (§3).
5. **Tag Prime Directive's curriculum skills** on merge (~10 min). Two gaps
   confirmed against `content/curriculum/`: no skill for *factors, multiples
   and primes* (nearest is a Y3 unit-fractions entry) and no *order of
   operations* at all. Both are UK Y6 blocks it builds locks on.
6. Visual regression check — screenshot each activity, fail on change.
7. Lint rule on hex colours and `font-family` outside a token block; make
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

Open items only, each with an owner. Failures already caught by a check, and
standing traps, are in `CLAUDE.md`. **Documentation does not fire at 11pm** —
proved when a session hit the documented apostrophe trap two commits after
documenting it.

| Failure | Symptom | Owner |
|---|---|---|
| A session commits to `main` mid-task | Another session's uncommitted files land in a commit that does not describe them | Website Infrastructure — candidate `doctor` warning on a dirty tree |
| Renormalising rewrites the working tree | Uncommitted edits silently revert; no error, no conflict | as above; cost Doc Manager `STATUS.md` on 2026-08-25 |
| `npm run skins` reads the DB, not content files | Reports a stale count | Website Infrastructure |
| A branch lacking the skin its activity needs | Renders unstyled; reads as a CSS bug | candidate `doctor` check |
| Shared CSS changed for one skin | Another activity's look shifts, unnoticed | §8.6 |
| Fixed-height child in a stretching flex parent | Surplus renders as chrome, invisible at small heights | §8.6 |
| Evidence images not under version control | Not in `public/images/operations/<slug>/` | §9 |
| A doc referencing a file that does not exist | Reader hunts for it, or trusts a spec never written | Doc Manager — candidate link check |
