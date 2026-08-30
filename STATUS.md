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
| **Operation Builder** | Prime Directive + the Case File skin | main | no — images next |
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

**Production is verified** — 2026-08-25, for the first time. `--prod` queried
the real host and returned `ok` throughout: 14 migrations, both activities,
three training banks, 70 skills, 14 tags. **This database matches the repo.**
`8839d38` fixed the check; the first honest run returned five `?` because the
rows predated content hashes, and a re-import stamped them. **Re-run after
every publish** — `?` is not `DRIFT`, it means unconfirmable.

A third deploy went out unplanned — a `docs` push carried WI's two commits
(§10); re-checked green after, but nobody decided it.

**Deployed repeatedly on 2026-08-25**, re-verified each time: pages 200, zero
`fonts.googleapis.com` requests, `--prod` green. **Prime Directive's art 404s
in production, correctly** — its content and images are on the branch, so none
of the Operation is live and the Case File CSS styles nothing yet.

**No manual step in any range** — checked, not assumed. Before any push run
`git diff --stat origin/main..main -- supabase/migrations/ content/`; both
fail silently when missed.

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

1. **A branch per session.** Doc Manager has one; the other two commit to
   `main`, where a commit ships whenever anyone else pushes — twice now. The
   pre-push check helps the pusher, not the author (§10).
2. **Gate `completion` on progress — Prime Directive cannot publish until it
   lands.** `state.ts:215` sends it unconditionally: at 0 of 7 phases the
   browser holds "ACCESS GRANTED… COGSWORTH" and `C-09`, Lock 07's answer.
   *(Website Infrastructure.)*
3. **Tag Prime Directive's curriculum skills** (~10 min). Two gaps confirmed
   in `content/curriculum/`: no *factors, multiples and primes* (nearest is a
   Y3 unit-fractions entry) and no *order of operations* — both UK Y6 blocks
   it builds locks on.
4. Visual regression check — screenshot each activity, fail on change.
   **Clear intervals and cancel animations before capture** or the HUD timer
   never lets the page idle (Op Builder proved this out). Carry a
   **rendered-width assertion per skin**: a known string in the display face
   against a nonexistent family, failing if they match.
5. Lint rule on hex colours and `font-family` outside token blocks; make `"import"` a glob.

## 9. Open decisions — waiting on Maciej

- **The Drive "Accounts" doc holds live credentials in plain text** — a
  database password and a mail-provider key; one `service_role` key has
  already been rotated this week after a transcript leak.
- **Local DB has Prime Directive `published` as `BB-0009`**, repo says
  `draft` — local only; `deploy:check` flags it `?`, correctly.
- **Duplicate style templates** in Maciej's image folder; `content/styles/`
  is authoritative. Unresolved across three revisions.
- **Nine suspect portraits stay outside version control**, deliberately —
  `doctor` warns about art nothing references, and they belong to a Suspect
  Log panel that does not exist yet. The seven referenced are in `9405d9c`.

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
| The apex redirects, so a bare status check misreads it | `thebrainbureau.app` answers 308 to `www.`; a naive uptime check reading 200-or-fail would call a healthy site down | follow redirects, or check the `www.` host |
| `completion` always serialised | `state.ts:215` sends every activity's ending on first load. **Prime Directive's ending is its answer** — COGSWORTH and `C-09` readable at 0 of 7 phases; Zero Hour survives only because its ending does not contain its answer | **§8.2, blocks publication** · Website Infrastructure |
| `Mission.tsx` hardcodes one activity's fiction for all | Every activity shows "⚠ ZERO HOUR" and completes on "VAULT SECURE" — the Value Vault is Zero Hour's story, inherited by a workshop short-circuit case | **unfixed** · Website Infrastructure; the content half went with `9405d9c` |
| A check that passes by matching nothing | `doctor` said "no activity references an image yet" of an activity referencing seven: the pattern was anchored to `/images/` and the paths were relative | **fixed** `b6f997b` — 0 matched before, 7 after |
| A guard everyone believes in that does not exist | Twice. This table cited `e2e` for `config._evidence`; those assertions were Zero Hour's own answers and would have passed while another activity shipped its deduction. And `designed: true` is believed to stop a half-drawn skin shipping, and gates nothing | **`e2e` real since `b6f997b`** — cite the assertion, never the script |
| A red suite with nobody watching | `npm run test` has failed since `735c6bb` — no CI, and the failure is a real question | **§8.1** |
| A safeguard whose exemption covers the case it was written for | `confirmRemoteWrite` correctly refuses a non-TTY run, then returns early on `process.env.CI` — set automatically by every CI, which is exactly where no human reads the warning | **§8.1** · Website Infrastructure |
| **Every cheap way of checking a font is wrong** | `getComputedStyle().fontFamily` returns what CSS *asked for*, so it said `VT323` regardless. `document.fonts.check('16px "VT323"')` returned **`true`** for a family not in the registry at all — it answers "would this be used", assuming system availability. `canvas.measureText` reported every family, real or invented, as an identical width. | **no check** · Website Infrastructure — the only honest test is rendered DOM width against a bogus family; candidate for §8.6 |
| Importers default to localhost when `SUPABASE_URL` is unset | `npm run import` writes to the laptop while appearing to publish | **§8.1** · Website Infrastructure |
| `npm run skins` reads the DB, not content files | Reports a stale count | Website Infrastructure |
| A branch lacking the skin its activity needs | Renders unstyled; reads as a CSS bug | candidate `doctor` check |
| Shared CSS changed for one skin | Another activity's look shifts, unnoticed | §8.6 |
| Fixed-height child in a stretching flex parent | Surplus renders as chrome, invisible at small heights | §8.6 |
| Evidence images not under version control | Not in `public/images/operations/<slug>/` | §9 |
| A doc referencing a file that does not exist | Reader hunts for it, or trusts a spec never written | Doc Manager — candidate link check |
