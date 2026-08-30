# Operation Prime Directive — what the mockup claimed

Written down after the fact, because the mockup was never committed and two of
its capabilities were rebuilt as something else. Recorded here so the third
time does not happen. See [README.md](README.md) for why this folder exists.

## Exhibits are drawn facsimiles, not magnified bitmaps

**This is the significant one, and it is not what shipped.**

Clicking an exhibit opens a **CSS-rendered object** — real DOM, real text, in
colours sampled from the bitmap. Not an enlargement of the image.

Each exhibit has its own drawn form:

| Exhibit | Drawn as |
|---|---|
| Power log | fan-fold printout with sprocket margins |
| Torque card | notched tag on an eyelet |
| Damaged badge | badge with photo box and corner scorch |
| Batch sheet | receipt torn along the bottom |
| Monitoring unit | screen with scanlines and a bezel |
| Parts trays | engraved metal plates |
| Radio swatches | swatch card of drawn chips |

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
- [ ] Per-exhibit forms as tabulated above

Unticked lines are `STATUS.md` §8 items. The activity does not publish while
any remain.
