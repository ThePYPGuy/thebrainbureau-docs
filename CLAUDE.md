@AGENTS.md

# Brain Bureau

See [README.md](README.md) first — architecture, the two rules that shape
everything (answer keys never reach the browser; students hold no database
session), and the recorded deviations from Activity Schema v0.4 under
"Deviations from Activity Schema v0.4". The schema document itself is not in
this repo — do not go looking for `activity-schema-v0.4.md`. This
file covers what the README doesn't: workflow, deployment, and gotchas
specific to working on this repo from outside it.

## Read the spec before building

[`docs/`](docs/README.md) is the product specification, and it outranks the
code. The README describes how this repo currently works; `docs/` describes
what it is meant to become. The two deliberately disagree in places, and
those places are recorded rather than quietly fixed.

Read [`docs/product-overview.md`](docs/product-overview.md) before touching
anything to do with activity types: Case, Operation, Agent Training and Field
Op have precise meanings, and the code still uses some of those words loosely.
Check the open questions at the end of [`docs/roadmap.md`](docs/roadmap.md)
before designing anything new — several foundations are decided but unbuilt.

## Before authoring a new activity, run `npm run skins`

Every Operation is designed standalone with its own look, and the failure mode
is quiet: several ship in a row wearing the same skin because it was the path
of least resistance and nobody counted. The script counts, names the untried
archetypes and what they suit, and says when the last three activities match.

**Tell Maciej what it says and suggest a different archetype.** He asked to be
prompted rather than to have to remember; the whole point is that this happens
without him raising it. The seven archetypes are locked in
[`docs/visual-identity.md`](docs/visual-identity.md) and enforced by a CHECK
constraint — an eighth is a design decision, not a slug you can invent.

## Working from Windows

This repo lives in **WSL2 Ubuntu**, not on the Windows filesystem. A previous
session lost an hour searching `C:\` for it. The actual paths, the remote and
the `\\wsl$` equivalents are in
[`docs/local/environment.md`](docs/local/environment.md) — machine-specific
values are kept out of this file so it can be mirrored publicly.

Run `npm`/`git`/`supabase`/`vercel`/`docker` inside WSL rather than from
Windows. From PowerShell: `wsl -e bash -ic '<command>'`. What is actually
installed, and where, is reported by `npm run doctor` — this file does not
say, because a written claim about an environment is wrong the moment the
environment changes and nothing announces it.

**Use `-ic`, not `-lc`.** A login shell resolves `npx` and `tsc` to the
Windows binaries on the shared PATH, which then fail in ways that look like
project errors rather than like the wrong executable.

Commit messages with an apostrophe can break through nested `bash -ic '...'`
quoting. Write the message to a scratch file and use `git commit -F <file>`
rather than fighting the quoting.

## Run `npm run doctor` first

```bash
npm run doctor
```

One line per check, PASS/FAIL/WARN, and the exact fix command on anything that
is not passing. Read-only — it reports, it never repairs. Under a second, with
a hard timeout on anything that touches the outside world, because a hang is
worse than a failure: a failure tells you something, a hang looks like
progress.

It exists because every expensive afternoon on this project has had the same
shape — **a description of the environment was believed instead of checked**:

- a doc said dependencies were installed with `uv sync`; `uv` was not on PATH
- a doc said `tools/google-image-gen` was cloned; the directory did not exist
- `vercel env pull` returned the literal string `[SENSITIVE]` instead of a key,
  and the auth failure that followed was reported as "dataset not found",
  which sent a whole session to fix the wrong thing

About an hour each. Doctor turns that archaeology into one line of output. It
asserts rather than describes: it reads PATH, the filesystem, git, and
`.env.local`, and prints what it found. Secrets are checked for shape — empty,
literal `[SENSITIVE]`, or truncated with `…` — and never printed; only a
prefix and a length.

**The division of labour: docs record decisions, doctor records state.** If you
find yourself writing a sentence in this file that asserts something is
installed, cloned, configured or present, it belongs in doctor instead, where
it will be re-checked every time instead of believed once.

Failures exit non-zero, so this can gate CI. Warnings do not: several are
conditions this repo is legitimately in, and a check that is always red is a
check everyone learns to ignore.

Related: `npm run deploy:check -- --prod` answers the other half — whether a
*database* matches the repo. Doctor is about this machine. Run it bare; it
finds the production credentials itself.

## Two agents, one repo — use a worktree

Only one branch can be checked out per folder. When a second session switches
branches, the first silently follows: it has no way to notice, and its next
commit lands on the wrong branch. That has already happened once — a homepage
commit landed on a feature branch and had to be cherry-picked back onto
`main`, and work touching a file that only existed on the other branch had to
be backed out.

Give each session its own folder on the same repository:

```bash
git worktree add ../tbb-prime-directive operation-prime-directive
```

Branches then cannot tread on each other and both sessions can work at once.
Which worktrees exist, and where, is in
[`docs/local/environment.md`](docs/local/environment.md) — and `git worktree
list` outranks it.

**Check before assuming.** `git worktree list` says what actually exists, and
`git branch --show-current` says which branch this folder is on. Run both at
the start of a session that might be sharing the repo.

## Deployment

**Run `npm run deploy:check -- --prod` after every publish.** Publishing has
three parts and only one is automatic: pushing to `main` deploys the app,
migrations do not run themselves, and content does not import itself. Both
manual parts fail *silently* -- the app reads the old row and carries on -- so
a feature can look deployed and be completely inert. It has happened twice: a
hint penalty nothing charged, and curriculum tags nothing could search.

The check compares a database against the repo and says what is out of step:
migrations not applied, content files edited but never imported, an empty
crosswalk, untagged tasks. It exits non-zero on real drift, and reports
"could not confirm" separately -- a draft that has deliberately never been
imported is normal, and a check that cries wolf gets ignored.

`--prod` resolves the production credentials itself, from the project linked by
`supabase link` plus that project's service_role key. **Do not pass
SUPABASE_URL and SUPABASE_SECRET_KEY by hand for a production check** -- the
former is unnecessary, and vite-node loads `.env.local` automatically, so a
hand-set production URL silently pairs with the *local* key.

It refuses rather than guesses. `--prod` with a localhost URL, a missing or
placeholder key, no linked project, an unreachable host, or a key the project
rejects all exit non-zero with the reason. And it proves the connection with a
real query before reporting anything, because "could not read" repeated across
every section used to add up to "nothing out of step".

The header prints the host it actually opened, derived from the same object
the client was built from. It cannot say one thing while querying another --
which it did, for a while, and that is why the rule below exists.

Content comparison works off a hash the importer stamps on the row
(`activities.content_hash`, from [lib/content-hash.ts](lib/content-hash.ts)).
It is canonical: key order, indentation, line endings and `_`-prefixed notes
do not count as changes, so only a real content edit shows as drift.

Push to `main` on `ThePYPGuy/thebrainbureau` and Vercel auto-deploys to
production at thebrainbureau.app. There is no staging environment and no CI —
verify locally first (`npm run setup && npm run dev`, plus the checks listed
in the README).

**Database migrations do not deploy automatically.** There's no workflow that
runs them — a new file in `supabase/migrations/` only reaches production when
someone runs `supabase db push` against the linked project. Pushing app code
that depends on a migration you haven't pushed yet will break production
silently (the deployed code, the old schema). Push the migration first, or in
the same breath as the code that needs it.

Project refs, the Vercel project name, the registrar and the mail provider are
in [`docs/local/environment.md`](docs/local/environment.md), along with the
note on why production Auth URL config is edited in the Supabase dashboard
rather than pushed with `supabase config push`.

## Traps that do not announce themselves

None of these produces an error. Each has cost this project real time. They
live here rather than in `STATUS.md` because they are properties of the system
rather than things anyone is about to fix — `STATUS.md` section 10 tracks only
what is still open, with an owner against it.

**Two sessions, one branch.** The worktree above separates branches, not
agents.

- **Commit by path. Never `git add -A`.** A session committing to `main`
  mid-task sweeps up whatever else is in the tree. `8839d38` is titled for a
  `deploy:check` fix and also carries another session's `CLAUDE.md` edits,
  committed by a session that had not written them.
- **Commit or stash before anyone renormalises line endings.** Adding
  `.gitattributes` in `f0563c2` rewrote tracked files from the index and
  silently discarded another session's uncommitted work — no error, no
  conflict, the files simply reverted to their committed state.
- **Read `git status` before committing.** Another session's work may be
  sitting in the tree, and it will not be labelled.

**Line endings** are normalised on commit by `.gitattributes` (`* text=auto
eol=lf`). The old rule — remember `newline="\n"` when writing from Windows —
is retired. It was easy to forget, and wrong when a file already had CRLF:
forcing LF then rewrote every line.

**Shared files are not Operation files.** `app/globals.css`, `lib/skins.ts` and
the engine render *every* activity, including whatever is already live. A
change made for one Operation changes the others.

The test is **what a rule can reach, not which file it sits in.** A block
scoped under `[data-skin="case-file"]` cannot affect Zero Hour, so it belongs
to whoever owns Case File even though it lives in the shared stylesheet.
Unscoped chrome — `.crtViewport`, `.plainViewport`, the token defaults — can
reach a live activity, so it is a platform change whoever writes it. Ask what
the selector matches, then decide.

**Skin tokens stay namespaced `--skin-*`.** `app/layout.tsx` binds
`--font-display` and `--font-body` on `<html>`, which sits above every
`[data-skin]` block. An un-namespaced token therefore loses silently, and
inherited prose wears the wrong face while headings look fine.

**A fixed-height child in a flex parent with default `align-items` stretches
the parent**, and the surplus renders as chrome. One such bug showed 167px of
extra bezel at a 873px window and nothing at all below ~746px, which is why it
survived review — the element itself was byte-identical and measured correctly.

**Schema and data.**

- New tables need an explicit `service_role` grant, or reads fail as a
  permission error that reads like a missing row.
- Entitlement changes need **code first**; schema changes need **migration
  first**. Getting it backwards fails silently.
- Static answers are numeric-only, so alphanumeric IDs need a prefix
  workaround until the engine handles them properly.

**A green check asserts less than it appears to.** `doctor` reads secret
*shape*, never identity: `sb_secret_… len 41` is byte-identical before and
after a key rotation. It catches empty, truncated and `[SENSITIVE]` keys, and
nothing beyond that. Know what a check actually claims before citing it as
evidence that something is done.

**Verifying a webfont loaded needs a rendered-width measurement.** Every cheap
check is wrong, and all three were believed here while the terminal rendered a
third oversized in a fallback face:

- `getComputedStyle().fontFamily` returns what the CSS *asked for*. It said
  `VT323` for a face that had never loaded.
- `document.fonts.check('16px "VT323"')` returned **true** for a family absent
  from the registry — it answers "would this be used", assuming system
  availability, not "did this load".
- `canvas.measureText` reported every family, real or invented, at an identical
  width.

The only honest test: render a known string in the target family and in a
deliberately nonexistent family, via the DOM, and compare widths. Equal means
the face is not loading. All faces now come through `next/font/google`, which
self-hosts — no render-time third-party request, and nothing for a school
network to block. Keep it that way rather than adding an `@import`.

**Image generation:** `--aspect` is ignored when `--ref` is passed, so the
output silently takes the reference image's dimensions.

**`npm run docs:sync` publishes to a public repo.** `STATUS.md`, `CLAUDE.md`
and `docs/` go; `docs/local/` never does. Assume anything written in those
three places is world-readable, because it is.

What `docs/local/` holds — project refs, deployment names, machine paths — is
kept back on a decision taken 2026-08-25. None of it is a credential; the
Supabase project ref already ships to every browser in
`NEXT_PUBLIC_SUPABASE_URL`. The test applied was not "is this safe to publish"
but **"does publishing it help"**, and it does not: a chat session sequencing
work never needs a project ref, and public git history is permanent.

It is held back by *not being copied*, never by redaction. A redaction step
that silently stopped matching its pattern would keep reporting success while
publishing the thing it was added to remove. Keep `docs:sync` a pure copy.

## Where things live

| | |
|---|---|
| Activities | `content/activities/*.json` |
| Training banks | `content/training/*.json` |
| Curriculum crosswalk | `content/curriculum/` |
| Image styles | `content/styles/` |
| Product spec | `docs/` |
| Machine-specific values | `docs/local/` — paths, project refs; not mirrored |

Operation plans and source data live in Maciej's own folders, outside the
repo; [`docs/local/environment.md`](docs/local/environment.md) says where.

Image styles moved into the repo on 2026-08-25. They had been outside version
control with nothing pointing at them — losing that folder would have lost the
house style with no error to say so. Anything the platform's appearance depends
on belongs where the platform is.

## Content is data

Adding or editing an activity (`content/activities/*.json` +
`npm run import`) doesn't need a migration or new components, as long as it
only uses task types the engine already knows. The importer matches on slug
and updates in place — it will not wipe existing student progress by
deleting and recreating phases/tasks. `npm run test:reimport` guards this;
don't remove that test when touching the importer.

**Content changes do not deploy with the code.** `npm run import` writes to
whatever database the environment points at, and Vercel does not run it. A
pushed content change is live in the repo and absent from production until
someone imports it — the same trap as migrations, with no error to announce
it: the app reads the old row and carries on.

Importing against production means pointing `SUPABASE_URL` and
`SUPABASE_SECRET_KEY` at the linked project for one command. The exact
invocation — written so the key never reaches the screen — is in
[`docs/local/environment.md`](docs/local/environment.md), together with two
ways of fetching that key that silently do not work.

The secret key bypasses RLS entirely and is the only credential that can read
answer keys. Don't paste it into anything that keeps a transcript; that has
already cost one rotation.
