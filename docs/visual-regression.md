# Visual regression — what it must cover and what will fool it

Deferred four times, three of them on Doc Manager's recommendation, on the
grounds that the surfaces kept moving. **Phase 3 landed and they have stopped.**

## Why it is not polish

It is the only check that would ever cover the drawn work — **sixteen hotspots,
twelve plate forms, seven glyphs**, verified by hand exactly once, by a person,
months ago. Every other check on this project passes while a page renders wrong,
because nothing *is* broken. It just looks it.

## The thing that changes the job

**`out/` is in `.gitignore`** — it is Next's static export directory. Every
capture taken so far lives on one machine and **nothing is committed**. A check
comparing against an uncommitted baseline compares against whatever is on the
disk of whoever runs it, which can only ever agree with itself. There is no
capture or diff script in `package.json` either, so this is a build from nothing.

**Pick a committed home** — not `out/`, not `.next/`, nothing the framework owns
or git ignores.

## Measurements already taken

Zero Hour, at 2560×1800, `deviceScaleFactor` 2:

| Region | Box |
|---|---|
| Status LED | `x 2080–2160, y 220–290` |
| Countdown | `x 1360–1640, y 290–380` |
| Blinking caret | `x 690–830, y 510–610` |

- **Masked: 0 of 4,608,000**
- **Unmasked: 3,144**
- **The page differs from *itself* by 977**, in the LED alone.

**That last number is the floor, and it has already caught something.** Any
harness reporting less than it on an unmasked self-comparison is not measuring
what it thinks it is — the first run to report a clean 0 was photographing an
empty CRT. See `CLAUDE.md`. Diff the page
against itself first, every run, and treat the result as the noise threshold.

## Three traps, each already paid for once

**Clear intervals and cancel animations before capture**, or the HUD timer never
lets the page idle. A raw diff first reported **169,606** changed pixels on Zero
Hour and it was the relocated boot sequence, not a regression.

**Chrome's checker imaging** makes two captures of identical code differ — 2,638
pixels inside the portraits, because Chrome paints a placeholder raster and
re-rasters when the decode lands. Waiting for decode rather than load got it to
1,032; a cache-warming reload made it **worse** at 2,413; forcing
nearest-neighbour worse again at 3,257. `--disable-checker-imaging` and the other
rasterisation flags took it to 0. **Only then does a zero mean anything.**

**`next/image` lazy-loads.** A full-page capture of a section the shutter never
scrolled to gets `naturalWidth: 0` and renders black — which looks exactly like
the change having failed. The tell that separates it from a stale image: **no
request was made at all**, so there is nothing in the network log.

## Carry a rendered-width assertion per skin

A known string in the display face against a nonexistent family, failing if they
match — with a **discriminating** string, since a 10px gap proves nothing. A
pixel diff catches a font that silently fell back; this explains it.

## Baselines

Phases two and three changed Bureau surfaces, so those want regenerating.

**`field-terminal.png` and `case-file.png` do not exist** *(checked 1 Sep — not
in either worktree, nowhere on disk, never committed)*. An earlier draft told a
session to preserve them, on a report Doc Manager recorded without verifying the
files were there.

**So the claim that three phases of visual work never touched activity chrome is
currently unevidenced.** Every baseline in `tests/visual/baseline` is dated 1 Sep
and post-redesign: it can prove *nothing changes from here*, and cannot prove
anything about what came before.

**It can be re-established, and that is a task rather than a footnote.** Check out
a pre-redesign commit into a temporary worktree, capture with this harness, and
diff against today. Until someone does, do not manufacture a replacement and do
not describe any baseline as predating anything.

**Say exactly what was regenerated.** A baseline accepted without being looked at
is a bug promoted to a standard.

## Its first real integration, 1 Sep

Twelve commits landed ahead of the harness's merge, including changes to
`app/globals.css` and `app/page.tsx` — both covered. **All eight surfaces still
matched, self-diff 0 on every one.**

That produced a result rather than an assumption: **the homepage renders
identically for a signed-out visitor now that `app/page.tsx` has stopped reading
the cookie.** Web Designer's half is pixel-neutral, evidenced instead of assumed —
the first thing this check has been able to say about somebody else's change, and
the reason it exists.

## Baselines are machine-specific, and only one machine here can make them

**Measured 1 Sep, on an unchanged page.** `/pricing` captured with Windows
Chrome at the committed baseline's exact viewport and scale differs from that
baseline by **326,418 of 10,163,200 pixels** — same dimensions, same layout,
**different rasterisation**.

**And it is not only rasterisation — it reaches layout.** Regenerated on Linux,
`pricing` came out at 4560, the same height Windows measured. `home` and
`profile` did not: 8042 against 7974, 3882 against 3828. **Different font metrics
move where text wraps, and the page height moves with it.** So the two platforms
do not render the same page differently; **they do not quite have the same
page.**

**A baseline therefore only means anything on the platform that made it.** The
bundled Chromium was chosen so a baseline reproduces elsewhere — true for
*version* drift, and it does not cross an operating system.

**Which makes regeneration a privilege, not a step.** `chrome-headless-shell`
dies in WSL on `libnspr4.so` without `sudo npx playwright install-deps chromium`,
and this machine has no passwordless sudo. **A session that cannot run the check
must not regenerate baselines from somewhere else** — Windows-rasterised PNGs
would break the check for everyone who can run it, and would look like a normal
update in the diff.

**What to do instead when you cannot run it:** hold the instrument constant.
Capture every surface **before and after** your own change with the same browser
and the same procedure. That measures what your change moved, which is the actual
question, and it does not touch the committed baselines. Website Designer did
exactly this and could report the four activity surfaces at 0 pixels.

## The capture path flakes, about one run in six

**Measured 1 Sep.** Global Intel Cards came back **212,321 pixels different
(4.61%)** on a commit that could not reach an activity. Diffing the page against
itself settled it: pre-change against current, **0**; five further captures
**byte-identical** at 456,984; the outlier against **both** sides, 212,321. **One
capture in six was a bad frame** — spread across the whole CRT area rather than
any element, invisible in a crop, and not reproducible on demand in four further
attempts.

**`scripts/visual-check.ts` shares that capture path** — same 3500ms wait, same
`quiet()` — **so `check:visual` can produce a false red on an activity.**

**Fixed 1 Sep (`99bb732`).** A failing surface now prints the baseline diff and
the self-diff **side by side, with a reading** — *the captures agreed exactly, so
this is a real change*, or the reverse. Both numbers are shown so the reader can
check the verdict rather than take its word.

**And the bad frame is caught, not merely explained.** When two captures
disagree, a third is taken and the two that agree are used: a bad frame disagrees
with both good ones while they agree with each other, which identifies it without
anyone knowing what causes it. **The discarded frame is always announced** — a
retry that hides itself is how a genuinely unstable page gets mistaken for a
settled one. If no two of three agree, it still fails.

**Proved by making both cases happen:** a one-channel `.brandPlate` edit went red
at 1,024px with a self-diff of 0 and read *real change*; a deliberately corrupted
first frame was discarded, announced, and the surface then passed. The injection
method is in the commit so anyone can repeat it.

**Unproven, and worth knowing:** nobody has seen this catch a *real* flake. It was
proved against injected corruption, and WD's bad frame was never reproducible on
demand. **Three captures is a judgement, not a bound** — two consecutive frames
bad in the same way would agree with each other and be believed.

## `profile.png` encodes database state, not just code

Its classes list renders **whatever deployments the local database holds** — two
or three per activity here, from seeding and re-minting, so each activity appears
twice. Real rows, not a rendering fault.

**So `profile` will go red after any re-seed, for a reason nobody changed in
code.** The activity surfaces do not have this problem: they render from content
JSON. **Left in and flagged rather than quietly dropped.** The weighing, from the
session that owns the harness: **do not give it a fixed fixture yet.** The cost
today is one regeneration after a re-seed — **visible and cheap**. A fixture is a
second definition of what a child's assignments look like, maintained by hand,
and it would drift from the seed **silently**: a loud small cost traded for a
quiet larger one.

**The question underneath is what this baseline is FOR.** Narrowing it — capture
`/profile` at a viewport that excludes the classes list, or accept it as a
*Bureau-chrome* check rather than a *content* check — answers that, and a fixture
does not.

**And it makes CI a real decision rather than an obvious win:** a CI runner is a
**third platform**, and its baselines would agree with neither of the two here.

## What it cannot see

**A third party that injects only a script is invisible to a pixel diff.**
`check:visual` went green on all eight surfaces when Vercel Analytics was mounted,
and that is not a pass — `@vercel/analytics` renders nothing. **The check could
not have caught it landing on a child's screen.** Served HTML is what answers
that question, and a green visual run should never be read as *nothing was
added*.

## Scope

**Three** activities are published — Zero Hour, Prime Directive and Global Intel
Cards — plus the Prime Directive print sheet. *(This file said two until 1 Sep.)*
The other archetypes do not exist. **Do not build a harness that assumes seven** —
drive it from a list, so a fourth is one line.
