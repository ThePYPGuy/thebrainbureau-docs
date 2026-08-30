# Operation Prime Directive — what the mockup claimed

Written down after the fact, because the mockup was never committed and two of
its capabilities were rebuilt as something else. Recorded here so the third
time does not happen. See [README.md](README.md) for why this folder exists.

## Exhibits are drawn facsimiles, not magnified bitmaps

**This is the significant one, and it is not what shipped.**

Clicking an exhibit opens a **CSS-rendered object** — real DOM, real text, in
colours sampled from the bitmap. Not an enlargement of the image.

The mockup defines **sixteen hotspots across eleven plate kinds**. Most locks
carry two or three exhibits, so there is no one-to-one mapping from lock to
form — an earlier version of this table assumed there was, and got three rows
wrong as a result.

Kinds, with the number of hotspots using each (counted from
`docs/local/mockups/operation-prime-directive-loupe.html`):

| Kind | Hotspots | Drawn as |
|---|---|---|
| `tag` | 4 | notched tag on an eyelet — the evidence tag |
| `label` | 2 | paper insert in a metal holder |
| `printout` | 1 | fan-fold printout with sprocket margins |
| `card` | 1 | cream calibration card |
| `sheet` | 1 | clipboard job sheet with a grid |
| `strip` | 1 | receipt torn along the bottom |
| `steel` | 1 | engraved metal plate |
| `hopper` | 1 | engraved metal plate |
| `glass` | 1 | screen with scanlines and a bezel |
| `radio` | 1 | swatch card of drawn chips |
| `badge` | 1 | badge with photo box and corner scorch |

Which lock uses which is in the source, not here — the mapping is many-to-one
and writing it out again is how it goes wrong again. **Read the loupe file.**

*Kind names and hotspot counts verified from the source. The drawn-form
descriptions are Op Builder's reading of the rendered plates, since the source
carries no prose describing them.*

Clicking opens the facsimile — **click, not hover**. And because the
facsimile carries the figures, the page does not repeat them: the on-page
exhibit transcript comes off once the drawn objects exist, **in that order**.

**There is no magnification at all.** That is the point rather than an omission.
Because the facsimile is text:

- it scales without blurring, at any size
- a screen reader reads the figures out
- it stays sharp on a projector, where a magnified sprite does not

The bitmap remains what the student sees in the case file. The facsimile is
what they get when they look closer.

### What shipped instead

`components/terminal/Zoom.tsx` enlarges the bitmap. Its own comment describes
the job as "enlarging an image... drawn at a fixed width and `pixelated`" —
built exactly to the instruction it was given, which asked for magnification
because nobody knew a facsimile had ever been designed.

The figures a lock turns on are carried as text elsewhere in the task, so no
lock is unsolvable and no reader is locked out. The loss is the reading
experience, not correctness.

## Capability list

- [x] Suspect Log: ten suspects, portraits, six attribute columns
- [x] Briefing open on load, typed on paper
- [x] Printable dossier
- [ ] **Exhibits open as drawn CSS facsimiles, not magnified bitmaps**
- [ ] Eleven plate kinds across sixteen hotspots, per the source

Unticked lines are `STATUS.md` §8 items. The activity does not publish while
any remain.
