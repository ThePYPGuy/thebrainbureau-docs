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

## Scope

**Three** activities are published — Zero Hour, Prime Directive and Global Intel
Cards — plus the Prime Directive print sheet. *(This file said two until 1 Sep.)*
The other archetypes do not exist. **Do not build a harness that assumes seven** —
drive it from a list, so a fourth is one line.
