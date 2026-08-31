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

**A brief handed to a fresh session must say where the repo is in its first
paragraph, with the `\\wsl.localhost\...` path spelled out.** This file explains
the arrangement and is no use for it — it lives inside the repo, so a session
that cannot find the repo cannot read it. Two sessions have now searched `C:\`,
found the spec folder under `Documents`, and accurately reported that the
project does not exist. A `~/tbb-...` path in a brief is worse than nothing on
the Windows side, because it looks like an answer.

Run `npm`/`git`/`supabase`/`vercel`/`docker` inside WSL rather than from
Windows. From PowerShell: `wsl -e bash -ic '<command>'`. What is actually
installed, and where, is reported by `npm run doctor` — this file does not
say, because a written claim about an environment is wrong the moment the
environment changes and nothing announces it.

**Use `-ic`.** Not `-lc`, and not a bare `-c`. `nvm` is sourced from
`~/.bashrc`, which only an *interactive* shell reads — so any non-interactive
shell has no `node` at all, `npm` falls through to the Windows `npm.exe` on the
shared PATH, and the failure arrives as a project error rather than as the wrong
executable.

Measured, not inferred: `bash -c 'npm run doctor'` in a worktree answers
*UNC paths are not supported. Defaulting to Windows directory*, then
*'vite-node' is not recognized*. The same command under `-ic` passes every check.
The trap is worth naming precisely because `doctor` is what you would reach for
to diagnose it, and `doctor` is the thing that cannot run.

Commit messages with an apostrophe can break through nested `bash -ic '...'`
quoting. Write the message to a scratch file and use `git commit -F <file>`
rather than fighting the quoting.

**The same trap applies to any text, not just commit messages.** A Windows
session driving WSL through `wsl -e bash -ic '...'` hands the string to a second
shell, so anything the *inner* shell treats as syntax is live: backticks run as
commands, `$(...)` substitutes, and a shell variable holding backticked text
expands when you echo it — which silently deleted a finished STATUS line here,
because the variable was captured from the file and written straight back.
`\x22` does not escape a quote for `sed`, and heredocs do not survive the nesting
at all.

The pattern that works, for anything with backticks, quotes or newlines in it:

```bash
# write the content with the editor tool, then
tr -d '\r' < scratch/thing.py > /tmp/thing.py && python3 /tmp/thing.py
```

The `tr` is not optional — a file written from Windows carries CRLF, and Python
will read `\r` into the strings it is matching on. Do the edit in Python with an
`assert` that the text you expected to find appeared exactly once, so a failed
match stops rather than silently changing nothing.

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

## Read STATUS.md before reporting on it

**Open it. Do not report a section as missing something you have not looked
for.** On 31 August one session reported four separate times that §4 carried no
production URL. It was at line 70 the whole time, and that session had not read
the file at all — every claim was sincere, none was checked, and each cost a
round trip to disprove.

The same session's other observations that day were accurate and valuable. This
is not carelessness; it is what reporting from a model of a file rather than the
file produces, and the failure is invisible from the inside because a
recollection and a reading feel identical.

**Cite by text, not by section number.** Numbers move — §8's items were
renumbered three times in one day, and two §10 rows spent weeks pointing at §8.3
and §8.4 after those items closed. The text survives a reorder; the number does
not. Quote the sentence you mean.

**Before saying a thing is recorded, read the line.** A `grep -c` returning 1
proves a word appears somewhere, not that your sentence is there — Doc Manager
signed off on a filed item that way and the match was an unrelated row.

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

**Re-read the activity's `_note` end to end before you publish.** Prime
Directive went out carrying four false sentences, and every one had been true
when it was written: not played yet, not tagged yet, not built yet, not made
yet. Notes are written while the work is unfinished, and nothing revisits them
when it lands — so publication is the last moment anyone looks at that file with
the whole build in view.

Delete each clause the work has overtaken. **Delete rather than reword to "done
now"**, which only starts a new clock on the same sentence; that the thing
exists is already visible in the app and the repo, both of which fail loudly if
it stops being true. Keep notes that explain a live constraint and name the code
enforcing it — those age slowly, and they are the reason the field is there.

To list the candidates rather than trust a skim:

```bash
grep -nEi "has not|not yet|does not exist|still (draft|untagged)|awaiting|later stage|for now" content/activities/<slug>.json
```

It finds sentences shaped like a claim of absence. It cannot tell you whether
one is still true — that is the read, and there is no script for it. See **A
note that records an absence has a shelf life the code does not** below for why
this class of sentence, and no other, needs the pass.

Editing a note is free at any time: it changes no hash and needs no import, so
correcting one on a live activity never touches production.

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

**Two column lists that must agree, in two languages.**
`scripts/import-activity.ts` builds the payload and the `import_activity()` SQL
function consumes it with explicit `INSERT` column lists. Add a column to
`activities`, `phases` or `tasks` in a migration, author it in the content JSON,
and the function ignores it — no error. Then `content_hash` is computed from the
*file*, so the hash updates and `deploy:check` reports that the database matches
the repo while the column sits empty. The old TypeScript importer had the same
shape with the list beside the payload in one file; it is now split across
TypeScript and SQL, and nothing audits the pair. When you add a column, change
both, and prove it by round-tripping a value through an import.

**A leak test will happily test somebody else's server.**
`test:answer-leak` fetches `BASE_URL`, defaulting to `127.0.0.1:3000`. Across
four worktrees that is whichever session started a server there first — which on
31 August was `tbb-platform`, not the worktree running the test. It passes, and
it has proved nothing about your branch. Before trusting a run, confirm the
server on 3000 is yours, or set `BASE_URL` to a port you started. The same shape
as the shared-working-tree hazard: the thing answering is not the thing you
think you are asking.

**A saved-HTML snapshot of a Field Terminal page is blank.**
`Frame.tsx` renders children only after a client-side boot effect, so any tool
that reads static HTML sees an empty terminal — and a "1 page, 0 images" PDF
looks like a pass rather than a blank sheet. Case File survives this because it
is plain chrome and server-rendered. Verify CRT pages in a live browser or not
at all.

**The browser pane reports geometry that is not there.**
Independently hit by two sessions on the same day. `innerWidth` comes back `0`
and `getBoundingClientRect()` near-zero — most often just after
`resize_window` with `preset: desktop`, which clears emulation — and it will
report a paragraph as 11px wide and hand back blank screenshots while the DOM
says the content is present. It also serves a **stale frame after a programmatic
scroll**: pixel-identical captures while the page has demonstrably moved. Pin an
explicit width and height before measuring anything, treat any measurement taken
while `innerWidth === 0` as void, and refresh by navigating rather than
scrolling. Both failures produce confident numbers rather than errors, which is
why they cost hours rather than minutes.

**The browser pane cannot reach the dev server on `127.0.0.1`.** The pane runs
on Windows and the server runs in WSL2, which forwards `localhost` but not
`127.0.0.1`. The pane then reports a *successful* navigation and renders a black
page — indistinguishable from a component that throws on mount, and it has
already cost one wrong conclusion. Use `localhost` from the pane. Note the
asymmetry: `scripts/` run inside WSL, where `127.0.0.1` is correct and is what
`BASE_URL` defaults to, so the two halves of this repo want different hosts for
the same server.

**The zoom overlay's skin is threaded by hand, at two mount sites.**
`Frame.tsx` mounts `ZoomProvider` at :98 and :107, each passing `skin={resolved}`.
The overlay is deliberately a *sibling* of the skinned viewport because
`.screenGlass` clips it, so the skin cannot cascade in. A third mount that
forgets the prop renders every drawn plate unstyled at 816px wide. Found that
way, not imagined.

**The Case File progress bar assumes cleared locks are a prefix.**
The `:has()` ladder takes the highest matching rung, which is correct only
because locks unlock in order. Out-of-order completion would under-report behind
a bar that still looks plausible. The CSS says so and nothing enforces it. The
dead end worth not rediscovering: a CSS counter can be read in `content` but not
in `calc()`, which is the entire reason there are twelve rungs rather than three
lines of arithmetic.

**`e2e-hint.ts` mutates a real task and restores it in a `finally`.** Killed
mid-run, that task keeps `hintPenalty: 40` in the local database. Local only, but
it presents as a mystery rather than as damage.

**A redaction pass cannot tell a secret from a protocol field.** Signal Check's
Sentry scrubber walked every string in the payload and redacted anything shaped
like a long run of hex — which is what a session id looks like, and also what
`trace_id`, `public_key` and `org_id` look like. Sentry answered **400 and
dropped every event**. From outside it was flawless: SDK installed, tunnel
answering 200, scrubber visibly running, and nothing arriving anywhere at all.

Found by reading a real envelope off the wire rather than by reading the
scrubber. It generalises past Sentry: **a pattern describing the shape of a
secret also describes the shape of the transport carrying it**, so a deep walk
over a payload you did not design will eventually redact the envelope. Redact
named fields you chose. Never everything that looks like the thing.

**Two Sentry config traps, both the same family.** `dataCollection: {}` is not
the same as `dataCollection` absent — the SDK branches on the key's *presence*,
so a block containing only comments enables `userInfo`, cookies, both header
directions, HTTP bodies in all four directions and stack-frame variables.
Present-and-empty is *more* permissive than absent, which is the opposite of how
an empty block reads. And **`debug: true` prints nothing in this repo**:
`next.config.ts` sets `treeshake.removeDebugLogging`, which compiles the logging
out. A diagnostic that silently does nothing is worse than one that is missing.

**There is one local Supabase stack, and every worktree shares it.**
`docker ps` shows a single `supabase_db_thebrainbureau`, not one per folder. So
`npm run setup`, `npm run db:setup`, `npm run db:test` and any bare
`supabase db reset` **wipe the local database out from under every other
session** — a worktree of your own is not a database of your own, and nothing
warns you. Ask before resetting while anyone else is working, and prefer
`npm run doctor` plus `npm run deploy:check` to find out whether your local
database is already fine, which it usually is.

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

**A token with a fallback, set by some faces and not others, fails invisibly.**
`globals.css` paints every `<strong>` with `var(--ink-strong, #ffffff)`. Field
Terminal and Case File set that token; the Bureau face never did, so emphasised
text fell through to the fallback and rendered **white on off-white** — the DOM
holding the text, the element having a box, `visibility: visible`, `opacity: 1`,
and only the pixels missing. `SimEditor.tsx`'s question heading and the class
panel's year label had been invisible on the dashboard for as long as they had
existed. Fixed at `e74e085` in `brand.module.css`, where the face is defined.

The fallback is the trap. Without one the text would have been unstyled and
obvious; with one it is confidently wrong, and every programmatic check agrees
it is fine. **When you give a token a fallback, check every face that does not
set it** — and this was found by looking at a screenshot and noticing that
"Answer:" had nothing after it, which is the third level in *Look at each
activity in a browser before pushing*.

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

**A diagnostic answers the question it was built for.** `content-fingerprint.ts`
was written to tell whether anything other than the named file moved during a
one-file import — a question where ownership is irrelevant, so it never
selected `owner_teacher_id`. Read later as an inventory of platform content,
where ownership is the *only* thing that matters, its output made a teacher's
draft quiz look like an orphan and nearly got it deleted. Every value it
printed was true. Before reusing a diagnostic to answer a new question, read
what it **selects**, not what it prints — and when you find the gap, fix the
output so the wrong reading is unavailable rather than merely discouraged.
That script now marks every row `platform` or `teacher <id>`.

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

**Except notes: editing a `_` key never needs an import.** `lib/content-notes.ts`
holds the `_`-prefix rule in one function, and four places call it —
`import-activity.ts`, `deploy-check.ts`, `content-hash.ts` and
`lib/server/state.ts`. The drift hash is taken *after* stripping, so correcting
a stale note on a published activity cannot drift production and needs no run
against it. The safe-feeling assumption cuts the wrong way: people either
re-import unnecessarily against a live database, or leave prose wrong because
they believe fixing it means touching one.

A fifth place deliberately does *not* call it, and must not:
`scripts/test-answer-leak.ts:96` keeps its own `startsWith` test on the `_`
prefix. The guard marks every string under an `_` key secret in the *authored*
file, then looks for those strings in the *served* state — so the two halves
must agree about nothing except whether the strip actually ran. Break
`stripNotes` today and the guard still marks `_evidenceDesign` secret, finds it
served, and fails. Make it import the shared function and the same break empties
its forbidden set: it passes, having tested nothing, on precisely the change it
exists to catch.

**A guard must not share its constant with the thing it guards.** This has bitten
twice. `docs-sync.mjs` grew a second gate derived from the first gate's list, so
adding a file to the allowlist published it cleanly through both; the fix was to
spell the rule out literally in `forbidden()`. The leak test is the same shape
one step from happening. Test a guard by **breaking what it guards** and
confirming it fails — a guard that goes quiet when its subject breaks was never
a second gate.

The trade is real, so this is a judgement and not a slogan. An independent copy
silently covers *less* after a deliberate convention change: move the prefix to
`__` and the copy keeps checking `_`. A shared constant falsely certifies the
*current* convention when it breaks. The second is worse — failing to cover
something new is visible the moment anyone looks, while reporting success on
something that regressed is not.

**A note that records an absence has a shelf life the code does not.** Prime
Directive published carrying four false sentences, and every one had been true
when written — not played yet, not tagged yet, not built yet, not made yet. A
note about something that exists ages slowly; a note about something that does
*not* is invalidated by the very work the project is trying to do, and nothing
fails when it expires. So: when you finish a thing, grep the notes for it — and
prefer deleting such a note to replacing it with *done now*, which only starts a
new clock on the same sentence. `lib/content-notes.ts` exists because its own
rule was written three times, one copy carrying the comment *the same way the
importer does* — true when written, with nothing to say so if it stopped.

Importing against production means pointing `SUPABASE_URL` and
`SUPABASE_SECRET_KEY` at the linked project for one command. The exact
invocation — written so the key never reaches the screen — is in
[`docs/local/environment.md`](docs/local/environment.md), together with two
ways of fetching that key that silently do not work.

The secret key bypasses RLS entirely and is the only credential that can read
answer keys. Don't paste it into anything that keeps a transcript.

**That has now cost two rotations, and the second was a different credential.**
The Supabase secret key went first. Then on 2026-08-31 a Sentry organisation
token was pasted into a chat while asking a question about the wizard that had
just printed it — rotated the same hour, and the new one went to Vercel without
passing through anything that keeps a log. So the rule is not about which secret
it is: a transcript keeps whatever reaches it, including the output you pasted
in order to ask what it meant. Rotate rather than reason about who saw it; it
takes a minute and the alternative is an argument with yourself.
