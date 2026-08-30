# Case File appearance — what the mockup claims

Source: `docs/local/mockups/case-file-appearance.html` (not published — see
[README.md](README.md)). It shows **the activity index**: folder, briefing,
lock list. Not a lock page.

## Type: two faces, and the reason is the point

| Role | Face | Used for |
|---|---|---|
| Typed content | **Special Elite** | the memo, prose, anything a typewriter produced |
| Printed labels | **Oswald** 500/600/700 | tab, stamps, headings, lock names, progress, case number |

The split is diegetic rather than decorative: **someone typed the memo; the
folder's labels were printed before anyone typed anything.** A single face
loses that, which is why the current build reads flat even now the paper is
right.

Both are on Google Fonts — verified, 200 — so both load through
`next/font/google` beside the others. Special Elite is already loaded.

> **This replaces Case File's current faces.** The tokens are Courier Prime
> (display) and Lora (body); neither appears in the mockup.
>
> **It therefore invalidates `--fd-scale`.** That is `.667`, derived as
> VT323-against-Courier-Prime and re-verified at 0.6671 by measurement. Oswald
> is a condensed grotesque, nowhere near Courier Prime's width, so all 17 size
> declarations multiplying through that scale would come out wrong — and
> wrong quietly, which is how the terminal ran a third oversized for four
> commits. **Re-derive the scale by measuring, do not adjust it by eye.**

## Palette

```
--paper       #d9c9a3     --ink         #3a2c1a
--paper-dark  #c9b688     --ink-soft    #6b5a3e
--stamp-red   #8c2f1f     --stamp-green #2f5d3a
--amber       #b3791a     --frame       #14171c
```

Two lighter papers sit on the folder: `#f3ead4` for the title box, `#faf6ea`
for the memo. Both carry ruled lines — a `repeating-linear-gradient` at 3px on
the folder, 22px on the memo.

Contrast must be measured, not assumed. `--ink-soft` on `--paper` is the pair
to check first, and `--amber` is the one that failed before at 3.54:1.

## Capability list

- [ ] Two-face type contract, `--fd-scale` re-derived by measurement
- [ ] Folder: ruled paper, tab offset from the left edge, deep shadow
- [ ] **ACTIVE CASE** stamp, rotated, translucent, top right
- [ ] Case progress bar — "6 OF 7 LOCKS CRACKED", green fill
- [ ] Title box on lighter paper; case number in stamp-red
- [ ] Briefing as a **memo on its own sheet**, rotated ~-0.6°, ruled, with a
      drawn **paperclip**
- [ ] Lock rows: per-lock **SVG icon**, number, name
- [ ] **CLEARED** stamp per solved lock, green, rotated ~-4°
- [ ] **NEXT →** tag on the active lock, amber, row tinted and outlined

Unticked lines are `STATUS.md` §8 items. The activity does not publish while
any remain.

## Not covered by this mockup

The lock page, the Suspect Log table, and the exhibit facsimiles — those are in
`operation-prime-directive.md`. This one is the index only.
