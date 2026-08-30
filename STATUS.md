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
| **Operation Builder** | Prime Directive + the Case File skin | main | no — Stage 2 next |
| **Website Infrastructure** | Platform, engine, scripts, checks | main | no — `914d97b`, `48a23b3` committed |
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
| Case File skin | `main` | none | Stage 1 done; Stage 2 next |
| Prime Directive | `operation-prime-directive` | none | merged to `2bf4df5`, already 4 behind |
| Platform | `main` | none | measure and CI exemption landed |
| Docs | `docs` | `STATUS.md`, `CLAUDE.md` | own worktree; merges to `main` `--ff-only` |

## 3. Overlap risks — READ BEFORE ASSIGNING WORK

Live collisions only; standing hazards are in `CLAUDE.md`.

- **A branch merge is a snapshot, not a subscription.** `2bf4df5` put the Case
  File skin and `doctor` onto `operation-prime-directive`; `main` moved again
  the same evening and the branch is already 4 behind. Run
  `git rev-list --left-right --count main...HEAD` before judging how anything
  on a branch looks — never the date of the last merge.
- **`package.json` `"import"` is a hardcoded list, not a glob.** Every new
  activity conflicts on merge; resolution is always "keep both sides". The
  `import:curriculum` conflict was already resolved by merge `46d7037`.
- **`bb49a62` duplicates `5fda3d3`** — same message on both. The duplicate is
  verified; cherry-pick residue is inference. Likely to conflict on merge.

## 4. Publish state

Pushing to `main` deploys the app. **Migrations do not run themselves. Content
does not import itself.** Both fail silently.

**Production is verified** — 2026-08-25, for the first time. `--prod` queried
the real host and returned `ok` throughout: 14 migrations, both activities,
three training banks, 70 skills, 14 tags. **This database matches the repo.**
`8839d38` fixed the check; the first honest run returned five `?` because the
rows predated content hashes, and a re-import stamped them. **Re-run after
every publish** — `?` is not `DRIFT`, it means unconfirmable.

A third deploy went out unplanned — a `docs` push carried WI's two commits
(§10); re-checked green after, but nobody decided it.

**Deployed 2026-08-25.** `d446fb4..eb46bfc` carried the Case File skin, the
`deploy:check` fix and the docs work; later pushes the self-hosted faces and
the importer guards. Re-verified each time: pages 200, `/dashboard` 307,
**zero** `fonts.googleapis.com` requests served, `--prod` green.

**No manual step was needed in any range** — checked, not assumed. Before any
push run `git diff --stat origin/main..main -- supabase/migrations/ content/`;
both fail silently when missed.

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
| Publish drift | `npm run deploy:check [-- --prod]` | built (`0760a88`); `--prod` fixed (`8839d38`) — exits rather than checking the wrong database |
| Line endings | `.gitattributes` | built (`f0563c2`) |
| Skin variety | `npm run skins` | built — reads the DB, not content files (§10) |
| Tests | `npm run test`, `npm run e2e` | vitest suite; 6 e2e scripts, incl. answer leak |
| Importer safety | `npm run test:reimport` | built |
| Targeted | `test:dashboard`, `:signup`, `:school`, `:entitlements`, `:curriculum` | built, 5 scripts |
| Docs mirror | `npm run docs:sync` | built |

**Activity schema is locked at 0.4**, declared by both shipped activities.
`activity-schema-v0.4.md` **has never been written**; `CLAUDE.md` says so.

## 7. Recently completed

Newest first. Hashes and order from `git log`; descriptions are each session's
own account of its own work, not re-measured here.

- **Doc Manager moved to its own worktree** on branch `docs` — the last shared-tree collision risk closed structurally rather than by care.
- `48a23b3`, `914d97b` — Field Terminal measured at 72ch (`ch` is honest there — Share Tech Mono's `0` and average glyph both 8.64px), callouts constrained from 101; `CI` exempts nothing. Both **live** — §10. *(WI)*
- `f04c924` — both viewports centred; Case File given a measure and a monochrome padlock. Settled by measurement at 1000px and 660px; the clipping objection does not apply because `.crtViewport` uses `min-height`. **Zero Hour confirmed centred by eye**; Case File's half not yet looked at. *(Op Builder.)*
- `a50672b` — **the skin faces were never loading**; the terminal had drawn in bare monospace, ~⅓ oversized, since sizes are tuned to VT323's narrowness. All four now self-host via `next/font`. `--fd-scale` re-verified at 0.6671 against rendered widths — the 0.667 guess held to four decimals, and could not have been checked before. Method in `CLAUDE.md`. *(WI)*
- `fc6cffb` — importers refuse to write blind: no localhost default, and a confirmation before any remote write. *(WI)*
- **13 commits deployed** (`d446fb4..eb46bfc`) — first push since the Case File work began; no migrations or content in the range, so nothing manual followed.
- **Production re-imported and verified clean** — first confirmed match between the deployed database and the repo.
- `b0ead55`–`5eff8d6` — `STATUS.md` into the repo, the allowlisted public docs mirror, and machine-specific values split into `docs/local/`.
- `735c6bb` — Case File skin, Stage 1 of 5: type tokenised, two contrast failures fixed.
- `b53ddf2` — Prime Directive: 7 phases, 7 answer keys, 6 hints. On its branch.

## 8. Next up

1. **Case File's `.calloutBox` has the 101-character gap** Field Terminal just
   closed — it covers `.preLine` and `.panel > p` but not the callout.
   Constrain the box, not its text. *(Op Builder, Stage 2.)*
2. Case File Stages 2–5 — dossier chrome, evidence capability (the one that
   matters), persistent panel, two correctness fixes.
3. **Re-merge `main` into `operation-prime-directive`** before judging how it
   looks — 4 behind again (§3).
4. **Tag Prime Directive's curriculum skills** on merge (~10 min). Two gaps
   confirmed against `content/curriculum/`: no *factors, multiples and primes*
   (nearest is a Y3 unit-fractions entry) and no *order of operations*. Both
   are UK Y6 blocks it builds locks on.
5. Visual regression check — screenshot each activity, fail on change. It must
   freeze or mock the clock (**the HUD timer never lets the page go idle**, so
   capture fails), and carry a **rendered-width assertion per skin**: a known
   string in the display face against a nonexistent family, failing if they
   match. That is the only check that would have caught the fonts.
6. Lint rule on hex colours and `font-family` outside token blocks; make `"import"` a glob.

## 9. Open decisions — waiting on Maciej

- **The Drive "Accounts" doc holds live credentials in plain text** — a
  database password and a mail-provider API key. A `service_role` key was
  already rotated once this week after landing in a transcript.
- **Local database has Prime Directive `published` as `BB-0009`**, repo says
  `draft` — Op Builder, to open it locally; nothing reached production.
  `deploy:check` flags it `?`, correctly, since the file is on a branch.
- **Duplicate style templates** in Maciej's image folder; `content/styles/` is
  authoritative. Unresolved across three revisions.
- **Evidence images outside version control** — 6 PNGs, beside a
  `_superseded/` folder of 7 older ones under near-identical names. Stage 3
  copies the 6, not the directory; a superseded image renders perfectly.
- **Case File is `designed: true` with Stage 2 of 5 to come** — does the flag
  mean "drawn enough to ship" or "finished"? (`lib/skins.ts:46`)
- **Merge `operation-prime-directive` now, or wait for evidence capability?**

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
| A docs push publishes every session's unpushed commits | `git push` pushes the branch, not your commits. Doc Manager pushed `914d97b` and `48a23b3` to production as a side effect of publishing `STATUS.md`; a deliberate hold would have broken silently | **unfixed** — check `git log origin/main..main` before pushing, and say what is going with you |
| Escapes written into a commit message by the shell that builds it | `printf "%s"` does not expand `\x27`, so six commits carry a literal `\x27` where an apostrophe belongs — permanent, and invisible unless the message is read back | **fixed** — write the message as a file, never assemble it in a shell |
| A rule contradicting the tables in the same document | §1 called three sessions Active while the line below forbade it and §2 put all three in one folder; every reader resolved it differently and none was warned | **fixed** — §1 now states an action and a check, not a state |
| A ticking clock blocks screenshot verification | The HUD timer re-renders every second, so the page never reports idle and capture fails; a session can measure geometry and never see the result | **§8.7** — unfixed for automation; `f04c924` was closed by a person looking instead |
| A safeguard whose exemption covers the case it was written for | `confirmRemoteWrite` correctly refuses a non-TTY run, then returns early on `process.env.CI` — set automatically by every CI, which is exactly where no human reads the warning | **§8.1** · Website Infrastructure |
| **Every cheap way of checking a font is wrong** | `getComputedStyle().fontFamily` returns what CSS *asked for*, so it said `VT323` regardless. `document.fonts.check('16px "VT323"')` returned **`true`** for a family not in the registry at all — it answers "would this be used", assuming system availability. `canvas.measureText` reported every family, real or invented, as an identical width. | **no check** · Website Infrastructure — the only honest test is rendered DOM width against a bogus family; candidate for §8.6 |
| Importers default to localhost when `SUPABASE_URL` is unset | `npm run import` writes to the laptop while appearing to publish | **§8.1** · Website Infrastructure |
| `npm run skins` reads the DB, not content files | Reports a stale count | Website Infrastructure |
| A branch lacking the skin its activity needs | Renders unstyled; reads as a CSS bug | candidate `doctor` check |
| Shared CSS changed for one skin | Another activity's look shifts, unnoticed | §8.6 |
| Fixed-height child in a stretching flex parent | Surplus renders as chrome, invisible at small heights | §8.6 |
| Evidence images not under version control | Not in `public/images/operations/<slug>/` | §9 |
| A doc referencing a file that does not exist | Reader hunts for it, or trusts a spec never written | Doc Manager — candidate link check |
