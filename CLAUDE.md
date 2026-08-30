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

**Write that file the way you would write any other file — do not assemble it
in the shell.** Escaping the apostrophe instead is how six commits here ended
up carrying a literal `\x27`: `printf "%s"` passes escapes through rather than
expanding them, and nothing complains, because a commit message is never
parsed. `-F` was never the weak point; building its input with shell quoting
was. Read the file back before committing if you did anything clever to
produce it.

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

**A shared working tree is the hazard, not a shared file.** All four collisions
on this project shared a tree; none needed a shared file. Two had provably
disjoint file sets and destroyed work anyway:

- `8839d38` swept another session's uncommitted `CLAUDE.md` into a commit
  titled for a `deploy:check` fix. The two sessions were editing
  `scripts/deploy-check.ts` and `CLAUDE.md`.
- `f0563c2` renormalised line endings and silently reverted another session's
  uncommitted `STATUS.md` and `scripts/docs-sync.mjs`. No error, no conflict —
  the files simply returned to their committed state.

Neither mechanism reads authorship. `git add -A` stages what is *present*; an
index-driven rewrite touches what is *tracked*. Working on different files
protects you from merge conflicts and from nothing else.

So the rule is about the tree, and it is an action rather than a state.
**Before your first write in a shared worktree:**

```bash
git status --porcelain
```

- **Empty** — the tree is yours. Work. Commit only the paths you authored, by
  name: `git add <paths>`, then `git commit -F <file>`. Never `-A`, never
  `-a`, never `.`.
- **Not empty** — stop. Do not edit, commit, stash, renormalise or switch
  branches. Name the files you found, say they are not yours, and wait.

Say so when you commit, so a session that is waiting knows the tree is free.

**`git push` publishes the branch, not your commits.** Anything another
session has committed and not pushed goes with you, and deploying is
automatic. Before pushing, look at what you are about to send:

```bash
git log --oneline origin/main..main
```

If it holds work that is not yours, say so before pushing, or wait. A
deliberate hold — commits kept back until a related change lands — looks
identical to work that simply has not been pushed yet, and breaking one
produces no error.

**These need an empty tree before they are safe to run at all** —
`git add --renormalize`, `reset`, `checkout .`, `stash`, and any branch
switch. Each rewrites files you did not author and reports nothing when it
destroys someone's work.

**A session that holds uncommitted work across another session's turn wants
its own worktree, not more care.** Git refuses the same branch in two
worktrees, so such a session needs a branch of its own.

**Each worktree needs its own `npm install`.** Turbopack rejects a symlinked
`node_modules` outright, so sharing one between worktrees does not work.

Doc Manager has one: `/home/maciej/tbb-doc-manager` on branch `docs`. It never
appears in `main`'s working tree, so nothing it holds can be swept into another
session's commit or reverted by a renormalise. Its cycle:

```bash
git -C ~/tbb-doc-manager merge main      # take main's history first
# edit STATUS.md / CLAUDE.md / docs/, commit there
git -C ~/thebrainbureau merge --ff-only docs
```

**`--ff-only` is the guard.** Only Doc Manager touches those three paths, so
publishing is always a fast-forward — it moves the ref and rewrites nothing
anyone else could be holding. If it ever refuses, the assumption has broken and
someone else has edited the docs; stop and look rather than forcing it. That
also makes publishing safe while another session is mid-task, which a plain
merge would not be.

Note the name. The mirror clone lives at `tbb-docs-mirror`, and a worktree
called `tbb-docs` beside it would differ by a suffix while being a completely
different thing — one the authoritative private repo, the other a disposable
clone of a *public* one. Committing in the wrong directory there fails
silently, so the worktree is `tbb-doc-manager` instead.

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

**Cite the assertion, never the script.** Two guards on this project were
believed in and did not exist: `e2e` was recorded as covering
`config._evidence` while its assertions only checked one activity's own answer
values, and `designed: true` was believed to stop a half-drawn skin shipping
while gating nothing. Before citing a check as evidence, read what it actually
asserts.

**A check can pass by matching nothing.** `doctor` reported "no activity
references an image yet" about an activity referencing seven, because its
pattern was anchored to `/images/` and the paths were relative. A rule that
finds zero things looks identical to a rule with nothing to find — so when you
write one, prove it matches something before trusting that it matched nothing.

**A green check asserts less than it appears to.** `doctor` reads secret
*shape*, never identity: `sb_secret_… len 41` is byte-identical before and
after a key rotation. It catches empty, truncated and `[SENSITIVE]` keys, and
nothing beyond that. Know what a check actually claims before citing it as
evidence that something is done.

**Commissioning an activity — the shape that costs least.** Learned across
Prime Directive, where most of the expense was avoidable:

1. **Ask for a playable-but-ugly version at about 30%.** Content and mechanics,
   default skin, no drawing. Play it. *Then* commission the look. The missing
   Suspect Log survived a full verification pass and was found in one sitting
   by someone playing — and the skin had been drawn twice around an activity
   that could not be played at all.
2. **Hand over the mockup first, as a file, whole.** Three instalments arriving
   mid-build each redrew surfaces already drawn.
3. **Answer the design questions in one go, before anyone starts.** Three faces
   not one, click not hover, the transcript comes off, screen *and* PDF — none
   was hard, and each cost a redraw for arriving late.
4. **Reuse an archetype unless the subject demands otherwise.** Drawing one is
   Stages 1–5 plus a redraw; wearing an existing one is content and tagging.
   `npm run skins` will still prompt for variety, and that is right in the long
   run — but a new archetype is roughly a whole build on its own.

**Look at each activity in a browser before pushing.** Three levels, and each
catches what the one before misses: computed styles missed the bezel bug
entirely, rendered geometry across window heights catches it, and a person
catches what neither does — the fallback fonts were spotted by eye after three
programmatic checks had all reported fine.

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
the face is not loading.

**The test is only as good as its string.** Special Elite measured 1352 against
a bogus 1342 — a 10px gap at ordinary size, which is not proof of anything. Set
large and pick characters the faces disagree about: at 200px the same test gave
1478 against 1310, decisive. Sanity-check what the bogus family resolves to
while you are there, since the fallback is what you are measuring against. All faces now come through `next/font/google`, which
self-hosts — no render-time third-party request, and nothing for a school
network to block. Keep it that way rather than adding an `@import`.

**Draw the replacement before removing the original.** Prime Directive's
exhibit transcript could only come off the page once the facsimiles carried
those figures as real text. Removing it first would have put the numbers beyond
a screen reader with nothing at all to report it, because the page renders
perfectly either way. Whenever one thing is to supersede another, the order is
not a preference.

**A zero from a fetch you have not confirmed succeeded is inconclusive, not
negative.** Checking the live stylesheet for markers returned zero for every
one — because the apex 308-redirects to `www` and the greps were running over
15-byte "Redirecting…" bodies. Confirm the fetch, then trust the count. Same
shape as grepping the wrong path and concluding the code is not there.

**`STATUS.md` is capped at 250 lines, and §7 is the only section that pays
for it.** The cap keeps the file readable in one sitting, which is the whole
point of it — but it must never be met by taking rows out of §10 or §8.

- **§7 is a window, not a record.** Oldest entries fall off. Git holds the
  history; this section holds what someone returning on Monday needs.
- **§10 is open items.** A row leaves when it is closed, or when its lesson
  moves to this file — never because the file is long.
- **§8 leaves when it is done.**

Both rules exist because both were broken. §10 was emptied a row at a time,
each removal defensible, watching the line count rather than the contents. §7
was trimmed to three entries and lost that week's playthrough, the production
import and two fixes — so a session reported from memory that a thing still
needed doing which had been done, because the file no longer said so.

**A prune that is hard to justify by contents is not a prune, it is damage
with a number attached.**

**A mockup is a claim, so commit it with a list of what it claims.**
`docs/mockups/` holds the mockup and a capability checklist beside it, and
every unticked line is a `STATUS.md` §8 item until it is built. Prime Directive
shipped without the Suspect Log and image zoom that a mockup had shown — not
lost, never here; `git log --all -S "suspect"` finds no such component in any
commit. It looked finished, so nobody checked whether it existed.

**Alt text can be the answer.** A portrait's description reaches the browser
like any other public string, and a child using a screen reader gets it in
place of the image. Describing one suspect as sheepish and another as calm
would hand over the deduction that the puzzle exists to make. Write every
description in one register — physical detail, no demeanour — so the alt text
carries what the image carries and nothing the image does not.

This applies to anyone writing alt text, not only to art sessions — a content
author writes it too, and will not have the skill loaded.
`.claude/skills/bureau-art/references/house-style.md` covers how to hit the
register while generating; the rule above is why it matters.

**Image generation goes through `.claude/skills/bureau-art/`.** It carries the
house style, the folder convention, and the half that has broken twice — read
the slug out of the activity file rather than inferring it, absolute paths,
`git add`, `doctor` as the finish line.

**The plugin ignores `--aspect` when `--ref` is passed**, so the output
silently takes the reference image's dimensions. That is still true of the
plugin and no longer true of the repo's generator, which builds one config on
every path and then measures the delivered file. Call the plugin directly and
the trap is still there.

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
