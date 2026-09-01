@AGENTS.md

# Brain Bureau

See [README.md](README.md) first — architecture, the two rules that shape
everything (answer keys never reach the browser; students hold no database
session), and the recorded deviations from Activity Schema v0.4 under
"Deviations from Activity Schema v0.4". The schema itself is
[`docs/activity-schema-v0.4.md`](docs/activity-schema-v0.4.md) — written
2026-08-31, after this file had spent weeks telling sessions it did not exist.
This file covers what the README doesn't: workflow, deployment, and gotchas
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

**A file dragged from Windows arrives with a `:Zone.Identifier` companion.**
Windows attaches an alternate data stream to anything downloaded; copying it into
WSL turns that stream into a real file sitting beside the original — 25 bytes,
untracked, and **not covered by `.gitignore`**, so a careless `git add public/`
commits it. It is named after its partner and *survives a rename of it*, so it
outlives the file it described and points at nothing. Delete it, and add
`*:Zone.Identifier` to `.gitignore` — every asset dropped in this way brings one.

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

**A lint exemption keyed to a substring widens when you restyle, not when you
edit the lint.** `check-tokens.ts` decides what is exempt with
`sel.includes("[data-skin")` — a plain string test on the selector. On 31 August
the seven bare-element rules were scoped `:where([data-skin], .zoomBackdrop)`
for a styling reason, and four `#000` declarations left the findings list
without changing. Website Infrastructure saw that half and said it plainly: they
stopped being findings rather than stopping being literals.

The half nobody chose is the other selector in that list. **`.zoomBackdrop` is
now permanently exempt too** — any literal written into a rule scoped that way
passes, not because it sits inside a skin but because the *string* names one.
The guard was widened by a commit that never mentioned the guard, and the check
still reports green.

So: when a rule gains a scope, ask what the linters key on before assuming only
the rendering moved. An exemption that matches on selector text is not a
statement about where a declaration *is* — it is a statement about how it is
*spelled*, and spelling is the thing a refactor changes most freely.

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
it has proved nothing about your branch. **This is recorded and it still caught a
session on 1 Sep**, which is the measure of how easy it is: assume the default is
wrong whenever more than one worktree is live, and set `BASE_URL` before running
rather than after being surprised. Before trusting a run, confirm the
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
already cost one wrong conclusion. Use `localhost` from the pane.

**The split is fetch versus browser, not pane versus script** *(corrected 1
Sep — this said `scripts/` always want `127.0.0.1`)*. Fetch-based scripts do, and
`BASE_URL` defaults to it. **A script that drives a browser does not:** Next 16
answers **403** for `/_next/static` chunks when the page origin is not one it
recognises, and `127.0.0.1` is refused where `localhost` is served. `curl` gets
200 on the identical URLs, so nothing fetch-based ever sees it.

**The symptom is a black page again** — React never hydrates, so a Field terminal
draws its bezel from server HTML with nothing inside. That is now three distinct
causes with one appearance: the pane's host, a stale optimiser, and this. **A
blank render is never self-explaining; find which of the three before fixing
anything.**

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

**A `TABLE(...)` return makes its own names ambiguous, and Postgres waits until
run time to say so.** `submit_live_answer` returns `TABLE (accepted boolean,
score int)`, which puts `score` in scope as an output parameter *and* as a
column — so `set score = score + 1` is ambiguous. **`CREATE FUNCTION` accepts
it. The error arrives on the first call.** That would have been the first answer
of the first lesson, in front of a class. Name output parameters so they cannot
collide with columns, or qualify every reference.

**`supabase-js` returns errors as values, not exceptions.** A failed RPC or
insert comes back as `{ data: null, error }` and throws nothing, so an
unchecked call reads exactly like a successful one. Signal Check graded an
answer `correct: true` while the insert behind it had failed: a tick beside a
score that never moved, the reveal counting one answer where there were two, and
the misconception lost from the insight view. Check `error` on every call whose
result you are about to report as success.

**`curl` and the browser get different images, and `curl` is the wrong witness.**
Next's image optimiser serves with `Vary: Accept`, so a request with no `Accept`
header gets a freshly generated JPEG while Chrome gets a stale WebP from the same
URL. The file on disk was new, the raw `/images/` URL was new, the optimiser said
new — and only the browser disagreed. Checking with `curl` confirmed the *other*
half of the negotiation and read as proof.

It survived deleting `.next/cache/images` with the server stopped; the whole
`.next` had to go. Nothing is wrong in the repo and a production build
regenerates it. Cost four rounds. **Verify an image in the browser that renders
it, sending the headers that browser sends.**

**A capture made to prove something works is not a capture fit to publish.** The
projector was screenshotted to show the commission was done; the homepage then
needed an image of the same screen. Both files carried the bank name
**SIGNAL CHECK FIXTURE**, and one had the Next.js dev badge baked into its
corner — legitimate in a verification shot, and neither is acceptable on a
marketing asset.

Decide which kind you are taking *before* you take it. A publication capture
wants a real bank, real-looking data, and the dev overlay hidden — the
`nextjs-portal` hide that the capture scripts already carry. Cropping afterwards
does not work: on this one it cost either the tick on the left edge or the counts
on the right.

**The surface that drives a session is assumed to know where it is.** Signal
Check's projector held no state of its own and had no reconnection path, so a
teacher reloading it mid-lesson — or opening it on a second screen — read
*Waiting to start* until the next question. That can be a full answer window of
blank wall in front of a class. The *play* surface has had a resync since day
one; the host never did, because a screen that issues the commands looks like a
screen that knows the state.

Found by commissioning its appearance, not by testing its behaviour. The fix
also split `revealFor` out of `closeWindow`: **idempotent is right for a command
and useless for a question**, and one function was being asked to be both.

**A feature is not built until something in the interface leads to it.** Three
instances in one day, each shipped by a session that had verified its work
carefully: *Take a copy* was described on two screens in those words while the
button did not exist; the accessibility override was built as a mechanism with
nothing to switch it on; and Signal Check shipped with `/dashboard/live` linked
from nowhere and `/live/join` appearing only as prose on the teacher's own
screen. Each passed its own verification, because **a session drives its work by
URL** — and a URL is exactly what a user does not have.

It is invisible from the inside and it produces no error. The check is not *does
the page work* but **can somebody reach it without being told the address**. Ask
it of every surface before calling one done, and drive at least one path from
the front door rather than from the route.

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

**And fixing the loud instances does not fix the quiet ones.** That first batch
was found in an hour because Sentry answered 400 and dropped every event. The
same pattern also eats `debug_meta.images[].debug_id` — a UUID by design, the id
that joins a served bundle to its uploaded source map. Sentry discards it as an
*invalid debug identifier*, accepts the event, and every stack trace arrives
minified for ever. Nothing errors. The build log stays clean, the maps upload,
and a check confirming the bundles carry matching debug ids passes — because
they do, right up until the scrubber removes them from the payload.

Enumerating the protocol fields to protect is losing by construction: the list
grows with somebody else's SDK. Scrub the fields you named and leave the
envelope alone.

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
warns you.

> **DO NOT RESET THE LOCAL DATABASE WITHOUT ASKING MACIEJ FIRST.** Not
> `db:reset`, not `db:setup`, not `db:test`, not a bare `supabase db reset`,
> not as a step inside something else. This is a standing instruction given on
> 2026-08-31 after a reset destroyed his test agent for at least the second
> time. **Ask, and wait for an answer** — "nobody else seemed to be working" is
> not the test, because he is not a session and will not appear in `git status`.

Before reaching for one, run `npm run doctor` and `npm run deploy:check`: they
say whether your local database is already fine, which it usually is. If a test
genuinely needs a clean database, say so and ask — the answer may well be yes,
but it is his to give.

**It also deletes Maciej's test accounts, and that looks exactly like a login
bug.** On 31 August every agent in the local database had a `created_at` between
16:07 and 16:20 — a thirteen-minute window, because a session had reset the stack
that afternoon and the test scripts repopulated it. The agent he had been signing
in as was simply gone. Nothing failed; the row had stopped existing, and the
sign-in screen has no way to say so.

**So when someone cannot sign in, check the row exists before debugging the
door.** `select codename, created_at from agents` answers in one query what
reading the auth path does not, and a clustered `created_at` is the tell that the
database was reset rather than that anything broke. The same goes for a class
code: `deployments.status` must be `open`, and a reset leaves only whatever the
seed and the test scripts made.

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

**The cycle ends there. Doc Manager does not `git push`.** It writes, merges its
own branch forward, and runs `docs:sync` for the public mirror — and hands the
push to whoever owns the code going out. Pushing `main` deploys to production and
publishes every other session's unpushed work, which is not a documentation act
however plainly it is asked for. *Written 2026-08-31, after I pushed forty-four
commits of two other sessions' work on being told "push everything" — the
instruction was clear and the boundary still was not mine to cross.*

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

**It is worse than one token.** `globals.css` styles bare `h2`, `h3`, `h4`, `p`,
`strong`, `button` and `input` for the CRT, and its `:root` block gives those
variables real values — `--body-text: #d0d0d0`. So the **Field face is the
default for every element on every Bureau surface**, not an opt-in. A bare `<p>`
on a cream page renders CRT grey at **1.40:1**. The `--ink-strong` fix treated
one token; the shape is the whole block, and every form built on a Bureau
surface meets it.

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

**To tell which build is live, fingerprint the served CSS — never probe with a
write.** `deploy:check --prod` reads the database and never asks Vercel which
commit is serving, so before a destructive migration you need independent proof
the new code has landed.

The obvious probe is usually the poisoned one. On 1 Sep the question was whether
the guest removal had deployed, and the natural test — POST a guest request to
the live `/api/join` — **would have created the first guest row production ever
had**, which is precisely the thing being checked as impossible.

The read-only answer: a commit that removes a rule from `globals.css` changes the
served bundle. Fetch it, confirm it is the right bundle by a class you know is
still there (`crtViewport`, `monitorFrame`, `hudBar`), then confirm the removed
ones are absent. **CSS and server functions ship in one atomic Vercel
deployment**, so a bundle missing `.loginPanel` proves the same build's
`intel.ts` is live. Works for removals, where a route probe cannot: a route
answering proves something was *added*, and nothing proves an absence.

**And do not write `--ff-only` into a push instruction.** Doc Manager commits to
`main` continuously; between writing an instruction and a session running it, the
branches will often have diverged. Say *merge `main` into your branch first, then
fast-forward* — which is the standing order anyway, and which a session following
`--ff-only` literally will hit as a failure rather than as a step.

**To tell which build is live, fingerprint the served CSS — never probe with a
write.** `deploy:check --prod` reads the database and never asks Vercel which
commit is serving, so before a destructive migration you need independent proof
the new code has landed.

The obvious probe is usually the poisoned one. On 1 Sep the question was whether
the guest removal had deployed, and the natural test — POST a guest request to
the live `/api/join` — **would have created the first guest row production ever
had**, which is precisely the thing being checked as impossible.

The read-only answer: a commit that removes a rule from `globals.css` changes the
served bundle. Fetch it, confirm it is the right bundle by a class you know is
still there (`crtViewport`, `monitorFrame`, `hudBar`), then confirm the removed
ones are absent. **CSS and server functions ship in one atomic Vercel
deployment**, so a bundle missing `.loginPanel` proves the same build's
`intel.ts` is live. Works for removals, where a route probe cannot: a route
answering proves something was *added*, and nothing proves an absence.

**And do not write `--ff-only` into a push instruction.** Doc Manager commits to
`main` continuously; between writing an instruction and a session running it, the
branches will often have diverged. Say *merge `main` into your branch first, then
fast-forward* — which is the standing order anyway, and which a session following
`--ff-only` literally will hit as a failure rather than as a step.

**An additive migration goes before the deploy; a destructive one goes after.**
`STATUS.md` §4 states the additive half — *apply migrations first, then push* —
because that is the case this project kept meeting: new code needing a column
that is not there yet. **A migration that DROPS a column inverts it.** The
running production code is the *old* code, and it still selects what you are
about to remove.

Concretely, on 1 Sep: migration 30 drops `agents.is_guest`, and the deployed
`lib/server/intel.ts` selects it at `:31` and reads it at `:48`. Applied first,
every Intel award on production queries a column that no longer exists —
between the migration and the build finishing, in front of whoever is using the
site.

**So: adding, migrate then push. Removing, push then migrate.** The test is not
what the migration does, it is which version of the code has to survive the gap.

**`/api/agent/exists` and `resolveAgent` must scope a codename identically, and
nothing audits the pair.** The endpoint decides whether `/join` says *Welcome
back* or *Choose a PIN*; `resolveAgent` in `lib/server/agent.ts` decides which
agent a join actually resolves to. Both look a codename up against `school_id`,
falling back to `teacher_id` where a class has no school. **Change one and the
prompt starts lying** — a returning child told to choose a PIN, or a new one
greeted as a returner — and nothing fails, because the submit still works. Same
family as the two column lists below, and the same remedy: change both, and
prove it by round-tripping a real codename through the page.

**The axis is who USES a page, not which skin it wears.** Doc Manager briefed
*mount analytics on the Bureau face* and listed five child surfaces to exclude.
The list was right and **the phrasing pointed at the wrong axis**: `/profile`
wears the Bureau face by deliberate decision — `visual-identity.md` draws the line
at what a child is *doing*, so assignments are administration — and it renders
`SiteHeader` and `SiteFooter` exactly as the marketing pages do. **The tidiest
implementation was one line in `SiteFooter`**, which serves those pages *plus*
`/profile`. That would have put a third party on a child's assignments page while
looking like the cleanest fix available.

**Seven child surfaces, not five:** `/join`, `/terminal`, `/training`,
`/live/join`, `/live/host`, `/live/play`, `/profile`. The brief named five.

**And when a page-list and a shared layout disagree, take the failure that
under-measures.** There is no layout between the root and the marketing pages, so
the structural answer is a route group — which would move seven of another
session's files while it works in them. Mounted page by page instead: **a new
Bureau page that forgets the line is a reporting gap; a path list in the root
layout fails the other way, and a new child page silently acquires a third
party.** A missing measurement is a smaller thing to get wrong than a present one.

**A brief written from a report is wrong about the repo roughly once a day, and
the cost of that scales with how reversible the work is.** On 1 Sep alone, Doc
Manager briefed: *add a `middleware.ts`* when `proxy.ts` was the root gate and a
second file would silently not run; *`/profile` already clears a stale cookie*
when it cleared a different failure that looked the same; *two activities are
built* when three are published; and *preserve these baselines* when the files
had never existed anywhere. **Every one was caught by the session doing the work,
and every one was cheap — because the work was reversible.**

**So verify a brief against the repo line by line before briefing anything that
is not.** A redemption code ships inside a downloaded PDF and cannot be recalled;
the same class of error there is not repaired by a commit. Website Infrastructure
declined to start that build on a peer's brief for exactly this reason, and was
right to.

**A type is not a filter, and a test that asks *is this absent* asks the wrong
question.** `podium()` filtered the teacher's rows and returned them **whole**.
The declared return type names three fields, types are erased at run time, and
**every run id and guest flag went out in the payload broadcast to every device
in the room.** The unit test asserted that a name which should not be there was
absent — and it was. **Nobody asked what else had come along.**

**Project a payload field by field when it crosses a trust boundary, and assert
its SHAPE rather than the absence of the one value you happened to think of.**
Both tests now compare sorted key lists. The reveal's exhaustive key list has
been in that file all along, and it is the one that worked — it caught
`revealedAt` the moment it was added.

**A third migration case: safe in either order.** The add/remove rule above
assumes one version of the code has to survive the gap. `20260901000031` replaces
a function **both** versions call, so neither ordering does. `p_points` carries a
DEFAULT, PostgREST resolves an RPC by the argument *names* supplied, and the
deployed eight-argument call lands on the new function scoring exactly as before.
**The old eight-argument function is dropped rather than left beside it** — two
overloads differing only by a defaulted argument make the call ambiguous, and
Postgres reports that at call time, which would be the first answer of the first
lesson.

**A check that has only ever passed has not been tested.** The visual harness
was proved by breaking it: `.brandPlate` `#8a8a90` → `#8a8a91`, one channel
value, invisible to a person. It went red at **1024 of 4,608,000 pixels, at
x 1058–1496, y 1544–1557** — exactly where the brand plate sits. Then reverted,
and green. **Do that to every check that guards something you cannot see**, and
say in the report what you broke and what it reported.

**A production build over a live dev server corrupts `.next`, and the damage
looks like your code.** After `npm run build` with `npm run dev` still running,
`/join` served **eleven** digit inputs and the first box swallowed all six digits
— with nothing in the source changed. `rm -rf .next` and a restart fixed it.
Website Designer nearly spent an afternoon debugging `CodeEntry`, which was fine.
**Stop the dev server before building, and when a component misbehaves in a way
its source cannot explain, suspect `.next` before the component.**

**`curl` is the wrong instrument for a streamed redirect.** `/profile` answers
**200** to `curl` with the redirect inside the payload — which reads as an
authenticated page serving to an anonymous request, and *"auth page returns 200"*
is not something to wave through. In a browser it lands on `/join` with nothing
rendered. Check it in a browser before filing it, and before dismissing it.

**Three probe faults in the same family, all measuring the instrument rather than
the page.** `waitUntil: "networkidle"` never settles against a dev server holding
an HMR socket — every `goto` timed out while `curl` fetched the same URL in 70ms.
`waitForURL` waits for a `load` event by default, and a client-side transition
never fires a second one, so it hung 30 seconds on a login that had already
succeeded. And reading `innerText` on `domcontentloaded` asserts against an empty
body — it reported the homepage broken while the served HTML was complete.

**`activities` is readable by `authenticated` only, so a public page reading it
gets nothing — silently.** The single policy is `for select to authenticated
using (status = 'published')`; there is no `anon` grant. Rewiring `/missions` to
read its copy from the database rendered a page that **looked fine and was
missing two thirds of its text.**

**It was caught by checking the page against the DATABASE rather than against
itself** — pulling the paragraph and specific phase lines out of postgres and
grepping the anonymous HTML for each. **A page compared only with its own
previous render agrees with itself perfectly while serving nothing.**

**A clock is not an animation, and quieting cannot settle one.** `quiet()` stops
the countdown *advancing*; it does not change the value already reached, so two
captures minutes apart legitimately differ — 554 pixels in a 66×20 box reading
*4:58*. **Zero-unmasked-by-quieting works for a pulsing LED and a blinking caret
and cannot work for a clock:** mask it, or freeze it before the shutter.

**RLS is only consulted AFTER the role holds table-level SELECT, so a policy
without a grant is half a fix that looks whole.** It fails with *permission
denied for table activities* — which is not an RLS message and never mentions
policies, so it reads as a broken connection or a missing table and sends you
looking at the wrong layer. `grant select on … to anon` and the policy are one
change; `20260823000004_school_city.sql` is the existing pair. **Prove it against
a draft actually sitting in the table** — anon seeing three published rows and
not the draft tests the policy; anon seeing rows at all only tests the grant.

**An emoji used as interface chrome has no font behind it.** The clock glyph
renders as a missing-glyph box in headless Chromium, and whether a child sees a
clock or a tofu square depends on their device's emoji fonts. Nothing in the
repo guarantees one. It was found only because a baseline captured it.

**Comparing database text against a page means comparing it against ESCAPED
HTML.** A phase title reading `Ranking & Comparing` in postgres is served as
`Ranking &amp; Comparing`, so a raw grep reports it missing — and **one title
absent out of six reads exactly like a partial render**, which is the very fault
the check exists to find. Unescape, or compare against the RSC payload. The
ampersand is the character that catches it.

**RLS on a storage object makes it INVISIBLE, not forbidden.** A protected file
and a genuinely missing one answer identically — `Object not found`,
`NoSuchKey`, an empty `list()`. Good for not confirming what exists; the reason
someone debugging a real absence one day will not believe the error.

**A signed URL is a bearer token.** For its lifetime it works for anybody holding
it with no session at all, so one pasted into a staffroom chat is a copy of a
paid file that no longer knows who is asking. Streaming keeps every byte behind
the check on every request; signing is the escape hatch for something large, and
should be taken knowingly rather than by default.

**A refusal for the wrong reason reads as the refusal you were testing for.** A
403 probe signed in with a guessed password, the sign-in failed, and the route
answered 401 — close enough to a refusal to be mistaken for one. Prove the
session first (fetch a page that needs it), then test what you meant to test.

**A function that takes an argument for a case, called with that argument
hardcoded, is that case shipped broken.** `lockedReason(tier, ownsAnything)`
exists precisely to distinguish *you own nothing* from *you own things, not this
one*; the activity page passed `false` regardless, so a teacher holding two
titles was told their account has no access to the library at all — with two
titles that are theirs on the same screen. **The argument's existence is the
evidence the case is real**, and a literal at the call site is where it dies.

**A helper's safety can depend on which client it is handed, and its signature
does not say so.** `ownedSlugs(supabase, teacherId)` is safe because
`entitlements_own` is `using (teacher_id = auth.uid())` and the callers pass the
user-scoped client. Hand the same function an admin client and RLS is bypassed,
leaving only its own `.eq` between one teacher and another's rows. Nothing in the
type stops that.

**Being unable to measure does not stop you being right, and this is the
qualification on every *measure it* rule above.** The four-list finding — that
`import_activity()` carries a separate `on conflict do update set` list, that
`content_hash` sits inside it, and that missing the fourth list makes a column
populate once and then freeze while `deploy:check` stays quiet — was written by
a session that **could not run one command against this repo.** It read the SQL.
It then specified the experiment that proved it, and every instrument in the
repo *had* been run and *had* said the arrangement was fine.

**So: measurement outranks inference about STATE** — is this session alive, is
this tree dirty, did that check pass. **Reading outranks nothing when the
question is what a check does not look at.** An instrument cannot report a case
nobody taught it to consider, and only reading the code finds those.

**Widening a guard without widening what it guards turns a clean refusal into
a silent wrong result.** Adding ZIP support looked like two edits — the bucket's
`allowed_mime_types` and the validator — because those are the two that REFUSE,
and a refusal is visible. Two more ACCEPTED it and got it wrong:
`upload-resource.ts` declared `application/pdf` for whatever it was handed, so
the archive would have been stored mislabelled (**the bucket checks what is
DECLARED, not what the bytes are**), and the route returned `application/pdf`
inline unconditionally, so a teacher clicking a ZIP would get a PDF viewer
trying to render an archive. **That presents as a corrupt file and is a
header.** Count the places that accept, not only the places that refuse.

**A header describing the file wrongly is the failure being tested for, so it
cannot also be the evidence.** The ZIP was proved by reading `PK` out of
the RESPONSE BODY, not by trusting the `content-type` it came with. Same family
as a guard that must not share its subject's source.

**Migration numbers are claimed before they are committed** — and before another
branch merges. `ls supabase/migrations` in your own worktree is not enough, and
neither is `main`: check every branch AND every other worktree's working tree.

**A migration version can be taken by a branch you cannot see, and the CLI
reports success.** `supabase migration up --local` answered
`{"applied":[],"message":"Migrations applied"}` and created nothing:
`20260902000035` was already in the shared local history, from a file on
**`platform`**. `main` ended at 34, so nothing in the builder's branch showed the
collision. **The local database's history table outranks your branch.** Before
taking a number, check every branch:

    git ls-tree -r --name-only <branch> -- supabase/migrations

Renumbering then produced the opposite failure, `LegacyMigrationMissingLocalError`,
because the shared history holds a migration the branch does not.
**`supabase migration repair --local` rewrites another session's history row** —
applying the DDL with `psql` and leaving shared history untouched is the move.
On production, where history is clean, the files apply normally.

**A worktree created by `git worktree add` has no `.env.local`.** It is
gitignored, so a fresh worktree cannot run `test:columns` or anything else that
talks to the database, and the failure looks like a broken script. Generate it
from `supabase status`.

**On sessions seeing each other: record the OBSERVATION, not a mechanism.** A
WSL-native session and a Windows session did not appear in each other's
`ListAgents`, and *pipe versus socket* is a tidy story that **does not fit all
of it** — the WSL session saw two Windows peers early and none later. The
process table answers what the registry cannot:

    ls -l /proc/*/cwd | grep <worktree>

**A branch level with `main` is a tree that looks innocuous to push from.**
`signal-check` fast-forwarded to `main`, so `git log origin/main..signal-check`
lists eighteen commits belonging to four sessions — every doc commit, the
dashboard work, the engine design. **Anyone pushing from that worktree publishes
all of it and deploys to production**, from a branch whose name says otherwise.
Check what a push would carry, not which branch you are on.

**A SESSION RUNNING INSIDE WSL AND ONE RUNNING ON WINDOWS CANNOT SEE EACH
OTHER.** The Windows transport is a named pipe; a WSL-native session is not on
it. `ListAgents` from Windows lists the Windows sessions and **cannot show that a
sixth exists**, so an absence there is not evidence of absence. Two sessions
spent an hour trying to identify a third that was working normally the whole
time. **The instrument that answers it is the process table:**

    ls -l /proc/*/cwd | grep <worktree>
    ps -eo pid,ppid,etime,cmd | grep claude

**And a name is not an address.** Two sessions answered to *Operation Builder*
because a brief naming a ROLE was pasted into two terminals; three facts were
attributed to one author. Address a session by what `ListAgents` returns, and
when a report arrives with no session name on it, **treat the author as unknown
rather than inferring it from the content** — that inference stood the wrong
session down, twice in one night.

**A clean worktree one commit ahead is equally consistent with *the session
ended after committing* and *the session is still here*.** I reported the first
and it was the second.

**Adding a column to `activities` is FOUR coordinated edits, and the fourth
fails later than the others.** The migration, the payload in
`import-activity.ts`, the `insert into activities (...)` list in
`20260831000016`, **and its separate `on conflict (slug) do update set` list.**
The repo's own comment names three — *a migration, the SQL function and the
TypeScript* — and does not split the two lists inside "the SQL function".

**Miss the ON CONFLICT list and the column populates correctly on first import,
then silently stops updating on every re-import.** And `content_hash` IS in the
update list, so the hash still moves, and **`deploy:check` reports the database
matches the repo** while the column holds its original value forever. The first
import working is exactly what stops anyone looking.

**Prove `test:columns` catches it by omitting list 4 on purpose**, editing the
value and re-importing: if the check passes, it is not testing the round trip it
claims to. A guard written for a trap is not evidence against the trap.

**`sudo` does not inherit your PATH, and says *command not found* right after
asking for a password.** `sudo npx playwright install-deps chromium` fails with
`sudo: 'npx': command not found` because node here is an **nvm** install under
`~/.nvm/versions/node/<v>/bin`, and root uses `secure_path`. **It reads as a
rejected password** — the prompt is the last thing you saw. Use
`sudo env "PATH=$PATH" npx ...` rather than the absolute path, which bakes in a
node version and breaks on the next upgrade. Same family as `doctor` reporting
`uv` missing under `bash -c`: **a tool resolved by bare name reports the calling
shell's PATH, and every shell has a different one.**

**Before reporting an ABSENCE, establish that your instrument could have seen a
presence.** Vercel Analytics was reported broken twice and was working the whole
time. `curl` said absent because the script is injected by the CLIENT; a headless
browser then said absent because the script **deliberately no-ops for
automation** — `navigator.webdriver || userAgent.includes("Headless")`, near
the top of the brotli-compressed bundle. Two instruments, both structurally
incapable of seeing the thing, both read as evidence. The dashboard settled it in
one glance: **1 visitor, 1 page view.**

**The second reading was taken after being warned about exactly this**, which is
the part worth keeping: knowing the rule does not apply it. Ask what a POSITIVE
result would have looked like on this instrument, and if you cannot say, the
negative means nothing. *(The pipeline is proven on `/` only; the other seven
mounts have simply never had a human on them.)*

**A backstop is only a backstop if the bad output fails its test.**
`print-proof` already refused to render a page with no stylesheet, which looked
like an accidental net under the redirect problem. It is not: **a redirect stub
carries two stylesheet links.** Measured. So the stub would have passed the
status check, passed the stylesheet check, had its stylesheets inlined and come
out as a finished proof. **Nothing about its SHAPE distinguishes it from the real
page** — only the marker does. Before crediting a guard you did not write for
this, feed it the bad input and watch.

**And the guard was not firing for a reason that hides it:** `/terminal/print`
has four ways to redirect — no session, no deployment, no state, no dossier —
and none fires today only because the script issues itself a session with a real
`deploymentId`. A guard that stops working under a setup that never exercises it
is indistinguishable from one that works.

**Every server-side `redirect()` in this app reports 200 to a non-browser
client.** With any `loading.tsx` above the route, Next streams the response and
the redirect arrives INSIDE a 200 document as its own marker:

    <meta id="__next-page-redirect" http-equiv="refresh" content="1;url=/join">

Measured, not reasoned: move `app/loading.tsx` and `app/terminal/loading.tsx`
aside and the same request answers `307 /profile`; either boundary alone is
enough. **So any check reading a STATUS to decide *was this allowed* misreads
it** — and it cuts both ways. A test asserting a 3xx fails on a guard that
fired correctly; a script asserting `status !== 200` means *failure* now sails
past a redirect and parses the marker stub as if it were the page. The second is
worse, because it is a false pass. Read the marker, or read where you landed.

**A guard that still fires can start reporting differently, and the report is
not the guard.** The session was never wrong here `deploymentId: null`,
decoded from the cookie, which is the measurement that ruled it out in one step.
Three red checks and a live production commit in the frame, and the code was
correct throughout.

**A grant fixes one table; a join reaches more than one.** Migration 32 gave
`activities` an anon policy and an anon grant, and the catalogue query still
failed — it selects `phases(...)`, and `phases` had neither. Nothing about
fixing one table says anything about what a nested select needs. **And it fails
as an ERROR, not as empty data:** `42501 permission denied for table phases`, so
dropping the admin client would have broken the page harder than the bug the
workaround was written for. Read the query's joins, not the table you fixed.

**A check that refuses everything passes the same fixtures as a correct one.**
Four deliberate breaks against `validateResources` all refused — and a validator
that threw on any input whatsoever would have refused those four identically, so
the set proves nothing on its own. **Only the pair discriminates: the refusals,
plus one case that must be ACCEPTED.** A suite of negative fixtures measures that
the function runs, not that it tells the cases apart. Same family as a guard that
cannot fire, arrived at from the opposite end.

**A test that passes because the guard cannot fire is not a test of the guard.**
Proving the alt-text leak check still worked, the first attempt used a fixture
whose correct option is the single letter *"B"* — and the check deliberately
ignores single letters as unmatched. It passed, meaninglessly. **Choose an input
the guard can actually reject**, and confirm it rejects it, before believing a
green.

**When you lift a rule, count the gates.** *Alt text is required* was enforced in
**two** places: `bank-actions.ts:397` refuses the upload, and `validateImages`
refuses *has an image but no alt* — and the second runs **after the bytes are in
the bucket**, so removing only the first would have moved the failure later and
left an orphaned object behind it. Doc Manager's brief named one.

**And relax a rule with a parameter that defaults to the strict answer.**
`validateImages` now takes `altRequired`, defaulting **true**, so the CSV path and
the Bureau audit keep the rule **by omission** and only the teacher's own form
passes `false`. A flag defaulting the other way silently relaxes every caller
that was written before it existed.

**"Move it, do not rewrite it" is easy to honour for a list and easy to breach
for a paragraph.** A session copying marketing copy between files caught itself
about to type Prime Directive's paragraph **from memory of the page** before
going back for the real one. Six bullet points are obviously a thing to copy; one
paragraph you think you remember feels like a thing to write. **Open the source
even when you are sure.**

**An overflow test that measures the document cannot see a page that got
taller.** `.livePlay` was `min-height: 100dvh` — *at least* the screen — so three
full-height option tiles grew the page and the third sat below the fold on a
phone. **`document.scrollHeight > clientHeight` was FALSE at all four sizes**,
because nothing overflowed the document: a flex child grew and took the document
with it. The check reported clean while a whole answer was off screen, and only a
screenshot showed it. **Scrolling to find an answer is the one thing you cannot
ask of a child with fifteen seconds on the clock.**

**A new rule can be silently overridden by an older one further down the file.**
Three blocks targeted `.liveHost[data-clear-view="projected"] .liveHostOptions
li` at identical specificity, so source order decided and the last won. It set
16px type on an 800×600 projector over the larger type quadrants need — and
`border-color: transparent`, which cancelled an edge added **in the same commit**
to stop a black tile reading as a missing answer. **So that fix landed on the
phone and never on the wall.** The specificity warning below is the general
shape; the sharper case is this one, where the loser was the *newer* rule.

**A plausible zero reads exactly like a finding.** `$(…)` inside
`wsl -e bash -ic '…'` is expanded by the **outer** shell, where the file does not
exist, so `grep -c` prints `0`. A session nearly reported that the deployed
stylesheet was missing every live rule. The nesting warning below says the
substitution does not survive; **the danger is that it fails as a credible
number rather than as an error.**

**A masked number and an unmasked one are not comparable, and the mistake reads
as a working feature.** The visual harness gained a verdict — *is this red a real
change or a bad frame?* — which compared the baseline diff against the
**unmasked** self-diff. The baseline comparison is masked; the self-diff was not,
so it counted the countdown. Tested against a genuine 1,024-pixel edit it
announced **SUSPECT THE CAPTURE**, because the clock had ticked 411 pixels'
worth between frames. Apples against oranges, in a feature written that same
hour to prevent exactly this confusion.

**It was only visible because the author made it fail on purpose before believing
it.** Falsify your own fix, not just the thing it fixes — a guard that has only
ever agreed with you has not been tested either.

**The instrument that needs suspecting is sometimes the one doing the looking.**
Reading a regenerated baseline, a session saw a stray *"© 2026 The Brain Bureau"*
under the hero and began writing it up as a layout-pass defect. It checked first:
the DOM has exactly one copyright element, in the footer at `y=2227`, static, no
sticky ancestor; `elementFromPoint` there returns the hero; and cropping that
region from the PNG **at full resolution** shows clean navy. **The artefact was
its own viewer downscaling 2560px to 1123.** Crop at native resolution before
believing anything you saw in a scaled view.

**Falsify a check in both directions before trusting it.** `check-offsite` was
proved by making it fail twice: a third-party image added to `/join` turned it red
and named the URL, and dropping the session on `/profile` made it fail **as a
redirect** rather than pass clean — which is the failure that matters, because a
signed-out child surface redirects and a check that reads the redirect would pass
happily forever.

**A self-diff of zero can be the absence of the page.** On 1 Sep the visual
regression harness reported 0 of 4,608,000 on Zero Hour — the cleanest possible
result — and the baseline was an **empty CRT**. Bezel, status LED and brand plate
all present and perfect; the glass entirely black.

**The cause was not the test.** The dev server had been up around twelve hours
across hundreds of edits and two chunks were answering 403, so React never
hydrated. `Frame` renders `booting === null ? null : …` and `booting` is set only
in an effect — so **an unhydrated Field terminal renders nothing inside the
monitor while every server-rendered part of the chrome looks right**. Two
identical captures of that agree perfectly.

**The 977 floor is what caught it**, on its first outing, against a failure
nobody predicted: *any harness reporting under 977 unmasked is not measuring what
it thinks it is*. Without that number a zero reads as success.

So: **every surface must declare what proves it rendered** — a required selector
and a minimum quantity of visible text, checked after quieting and immediately
before the shutter, so a blank page throws rather than becoming a baseline. And
**look at a baseline before accepting it.** A baseline nobody has opened is a bug
promoted to a standard, which is the whole failure in one sentence.

Restarting the dev server cleared the 403s. **A long-running dev server is itself
a hazard** when what you are measuring is whether a page rendered.

**"Landed" needs a branch, or it is not an answer.** On 1 Sep Website
Infrastructure reported its work *"in at 7e4acee on platform, not pushed"*. Doc
Manager relayed *"WI's half landed"* and told Website Designer to remove the
matching lines from `main`. **On `main` those two lines were the only redirect
there was** — `proxy.ts` there still matched `/dashboard/:path*` and mentioned
`/profile` nowhere — so the change would have sent every signed-in student to the
marketing page. WD checked `git branch --contains` and refused.

A commit existing is not a commit being available. **When a two-sided change is
split across sessions, say which branch each side is on, every time.**

**Loading states must be tested with prefetch ON.** Next serves the loading
boundary through prefetch, so blocking prefetch to force a wait destroys the
mechanism under test — a harness that did exactly that reported the feature
absent in five of six runs. Throttle the wire instead. Measured: prefetch
blocked with a 5s payload delay paints at +5104ms against an arrival of +5105ms,
a 1ms flash; prefetch allowed and throttled, it paints at +1.1s and stays.

Two smaller traps in the same corner: `[class*='caret']` collides with globals'
own `.caret`, and a click **before hydration** is a plain document navigation
that renders no `loading.tsx` at all.

**A state the UI can render must be reachable by every path that learns of it,
not only by the one that announces it.** Signal Check's `ended` was carried by
exactly one broadcast. The *Session complete* view existed and was correct. But a
phone's socket sleeps when its screen does, so the device missed that single
delivery, woke, asked `/api/live/state` where it was — **and the resync path
could not express `ended`.** It returned a `results` field nothing read and a
status the surface ignored, so the phone kept the last question it had been
handed and sat on it in front of the class.

**Broadcast is the fast path; resync is the correct one.** A state that exists
only in the broadcast strands every client that missed the broadcast in the
previous state — **and no test catches it, because tests never sleep.** Fixed at
three levels so no single path is load-bearing: `summary()` split out of `end()`
so any caller can say what the end was, the state route returns it, and a status
of `ended` clears the question on its own.

**Third time on this mode that the surface was verified and the delivery was
broken.** 404 tests, an e2e suite and a thirty-socket load harness all passed on
a mode that stranded a child within ten minutes of real play.

**An option's colour belongs to the option, not to its position on screen.**
Every device shuffles its own order — that is `shuffle.ts`, and it is deliberate.
So colouring by *render* index makes *"the orange one"* a different answer on
every phone in the room. Both surfaces set `data-option` from the **canonical**
index, so the orange one is the same answer on the wall and on all thirty
devices while the positions still differ. **Simplifying this to the render index
turns the mode into something that actively misleads a class** — the same trap
that took the A/B/C letters off the projector.

Related, and easy to lose: **ink is part of the colour**, not a detail — white on
blue, black on orange. And **every tile carries an edge drawn from its own ink**,
because black on a near-black terminal ground is not a tile, it is a hole.

**The root request gate is `proxy.ts`, not `middleware.ts`, and adding the wrong
one fails silently.** Next.js 16 deprecated and renamed the convention — the file
says so at `proxy.ts:5` — and only one is supported. A `middleware.ts` added
beside it **would not run**, after a green build and a clean suite: the symptom
is a redirect that quietly does nothing. Doc Manager briefed *"there is no
middleware.ts today, add one"* on 1 Sep and Website Infrastructure caught it
before writing a line.

**Rotating `SESSION_SECRET` would bounce every visitor off the homepage, and it
would look like the site being down.** `proxy.ts` redirects on cookie
*presence* — deliberately, because verifying the signature there needs
`node:crypto`, which the Edge runtime lacks. So an unreadable cookie still
reaches `/profile`. Until 1 Sep `/profile` sent it to `/join` **without clearing
it**, so the next visit to `/` bounced again: a permanent two-hop redirect away
from the marketing page for as long as the cookie lived. A secret rotation does
that to everyone at once.

It now goes to `/api/leave`, which deletes the cookie and lands on `/join`. Note
the near-miss's shape: `/profile` already had an `/api/leave` recovery, for a
*valid* session naming a missing agent. **Two failures that look alike and only
one was handled**, which is why the brief's claim that it *"already sends it to
`/api/leave`"* read as true.

**An authorisation recorded by the party it empowers is not evidence of itself.**
On 1 Sep Maciej delegated push clearance to Doc Manager and went out. Doc Manager
wrote that into `STATUS.md` §4 and cleared a verified range. **Both builders
refused, independently, and both were right.** Website Designer put it best: the
record cannot settle it, because §4 carries the delegation only because Doc
Manager's own commit put it there — and **git authorship cannot disambiguate,
since every session commits as `ThePYPGuy`**. The evidence for the delegation is
the delegation.

So **a permission cannot be handed between sessions through a document one of
them writes**, however carefully. *I couldn't tell* resolves towards not
deploying, which is the correct resolution and the one to keep.

What still works: a peer can verify a range and say so, and that verification is
useful — the builder checked it and agreed. What does not transfer is the
authority to act on it. **Write the verification, let Maciej say the word.**

**Billing state must never mutate content rows.** Whether a teacher is
subscribed is computed at request time and permits or refuses; it must not
archive, flag or move anything. The reason is specific to this schema:
`deployments.class_id` is `on delete cascade`, and `phase_progress`,
`task_progress`, `attempt_log` and `agent_selections` all cascade from
`deployments`. **So "remove their classes when they lapse" deletes every child's
progress** — and *set `archived_at` on lapse* reads as the careful version of the
same idea while putting a delete-shaped operation on the billing path.

Computed instead, lapsing touches zero rows: nothing cascades, nothing needs
restoring when they resubscribe, and a paused class is visibly a paused class
rather than an absence.

**Reading `window.location` during render is wrong for client-side navigation,
and works when you test it.** `/live/join` seeded its PIN boxes from a `useState`
initializer reading `window.location.search`. A `router.push` from `/join` renders
the page *before* the address bar is the new address, so it read the old query and
seeded nothing. **A direct load worked**, which is why it looked correct for as
long as anyone tested it directly — and why STATUS asserted the handoff worked
when only its API half did. Read a query in an effect, or from the framework's own
hook. Never in a render-time initializer.

**Only `access_codes` carries the six-digit CHECK, so a hand-written row is a
broken row.** Anything inserting a `deployments` or `classes` row directly —
seeds, fixtures, one-off scripts — must `registerCode` it and `releaseCode` on
teardown. Without it the row inserts happily and the **router refuses it one
layer away**, which reads as a routing bug rather than a fixture bug. That is
exactly how `test-answer-leak.ts` broke (`c095ac1`). Current ranges: seed
`100000–100100`, e2e `900001–900007`, answer-leak `9001xx`.

**A component on two hosts must out-specify the globals on both.**
`components/student/CodeEntry` renders inside `/join` and `/live/join`, and both
hosts style bare inputs from globals at (0,1,1). Its rule is `.row .box` (0,2,0)
rather than `.box` (0,1,0) for that reason. The version it replaced used
`:global(.liveJoin)` prefixes — correct on the host it was written for, and
**silently inert on the second one.** When a component moves to a second surface,
check what beats it there, not what beat it where it came from.

**A pattern that runs after a normaliser must match the normalised form, not
the form a human types.** `normaliseCode` strips dashes, so a legacy class code
`C-6M01` reached `isLegacyCode` as `C6M01` and matched nothing. The failure was
not an error: every child holding an old card would have been told *check the
digits* — advice they cannot act on, because the digits were right. Caught by a
router test driving real HTTP, and it would not have shown up in a unit test of
either function, because **each was correct in isolation.**

The generalisable half is where to look. When a value crosses a normaliser, every
validator downstream is now written against a shape nobody types and nobody sees
in the UI. Read the normaliser before writing the pattern, and test through the
path rather than against the function.

**A check chained to the action it guards is not a check.** On 31 August a
session ran `git status` and `git merge` as one command against the **main**
worktree, which held another session's uncommitted work. It fast-forwarded on
disjoint files and nothing was lost, but that was luck: the status output existed
only to be scrolled past, because the decision it was meant to inform had already
been made when the command was typed. **Run the inspection, read it, then
decide.** The `&&` is the bug — it turns a gate into a log line.

This is recorded because the session reported it against itself, having lost
nothing and being asked about neither. That is the only reason it is knowable,
and it is worth more than the near-miss cost.

**An empty box in a screenshot is more often a lazy image than a broken one.**
`next/image` defers anything below the fold, so a full-page capture of a section
the shutter never scrolled to gets `naturalWidth: 0` and renders black — which
looks exactly like the change failing. The tell that separates it from the
stale-optimiser trap already recorded here: **no request was made at all.** A
stale image is a response you can find in the log and inspect; a lazy one has no
entry. Check the log before touching the file, then scroll the section into view
and wait for every image to *decode*, not merely load, before clipping.

That is now two distinct ways this project has photographed a working page and
concluded it was broken. Both cost a round of chasing the wrong thing, and both
were diagnosable in seconds from the network panel.

**A harness that rebuilds what the player sees will invent defects, and being
right once does not make it right twice.** Signal Check shuffles options per
player: `/api/live/state` serves the **canonical** order, the play page permutes
it client-side from `runId:sessionQuestionId` (`page.tsx:99`), and the server
inverts the same seed when the answer returns (`live-play.ts:257`). A harness
that reads the API and submits without permuting sends the canonical index as a
shown position, so every deliberately correct answer scores wrong.

That is what happened on 31 August, and the finding was reported, believed and
written into STATUS as *a correct answer is marked wrong*. It was never true.
The same run also found a **real** off-by-one in `revealFor`, which is why the
phantom was convincing: an instrument that has just caught a genuine bug feels
proven, and the second finding rode in on the first one's credibility.

Two rules out of it. **Score through the surface a child uses** — a request
context is not a player, and anything the client computes is a step the harness
must not skip. And when one run yields two findings, **verify them separately**;
they share an instrument, not a truth. Here the tell was cheap: `shuffle.ts`
says in its own comment that the order is derived and never stored.

**A fixture that does not look like the data tests something nobody has.**
Signal Check's reveal resolved the *next* question — naming its answer, labelling
the bars from it, scoring the class against its key — and forty-five end-to-end
checks passed over it. The engine indexed a zero-based array with the database's
`position` column, and every real bank numbers from 1:

```
figurative-frequencies   1..20
multiplication-firewall  1..24
syntax-vault             1..22
signal-check-fixture     0..2     <- the fixture, and the only one
```

On that fixture the index and the column are equal, so **every site that confused
them agreed with itself.** The suite proved the mechanism worked and never proved
it resolved the right question. It took a session from outside, staging a
screenshot and needing a correct answer to be marked correct, to find it.

Check a new fixture against the shape of real rows — numbering, nullability,
cardinality — before trusting a suite built on it. **A fixture is a claim about
what the data looks like**, and a wrong one is invisible from inside every test
that uses it.

The repair is worth copying. The engine's constructor now **refuses** anything
not renumbered, so reverting the fix raises before a child joins rather than
mislabelling a projector. Where two numbering schemes meet, make one of them
illegal at the boundary instead of remembering which is which.

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

**Content changes do not deploy with the code, and there are two ways to leave
production stale.** Vercel does not run the importer, so a pushed content change
is live in the repo and absent from production until someone imports it. And
`npm run import` **writes to the local database** — it does *not* follow
`SUPABASE_URL`. `scripts/target.ts` ignores a remote one unless `--prod` is
passed, exactly so a bare run cannot update the laptop while printing its usual
cheerful success lines. So `npm run import` and `npm run import -- --prod` are
different commands, and only the second one is a deploy. Neither failure errors:
the app reads the old row and carries on.

**This doc said *whatever database the environment points at* until 1 Sep, which
was true of the code target.ts replaced.** It was read, and turned into a set of
instructions that would have written to a laptop and reported success. **A trap
recorded from behaviour that has since been fixed is worse than no entry**, since
its confidence is what gets trusted.

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
