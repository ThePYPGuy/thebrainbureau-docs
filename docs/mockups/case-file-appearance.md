# Case File appearance — what the mockup claims

Source: `docs/local/mockups/case-file-appearance.html` (not published — see
[README.md](README.md)). It shows **the activity index**: folder, briefing,
lock list. Not a lock page.

## Type: three faces, three roles

| Role | Face | Used for |
|---|---|---|
| Printed | **Archivo Narrow** | tab, stamps, headings, lock names, progress, case number |
| Typed | **Special Elite** | the briefing memo |
| Read | **Lora** | lock prose, anything read repeatedly |

The mockup uses Oswald for the printed role; **Archivo Narrow replaces it** —
the same condensed shape, quieter, closer to the gothic actually printed on
filing furniture and less recognisable as a web heading face.

The printed/typed split is diegetic: **someone typed the memo; the folder's
labels were printed before anyone typed anything.** A single face loses that,
which is why the surface reads flat even now the paper is right.

Lora is the third role and a legibility decision rather than an aesthetic one.
Special Elite is distressed and irregular — right for a memo read once, hard
work across prose a child returns to, particularly a dyslexic one.
**Atmosphere where it is read once; legibility where it is read often.**

All three verified on Google Fonts. Lora and Special Elite already load;
Archivo Narrow is the only addition.

> **This replaces Courier Prime**, the current display token. Lora stays, in a
> narrower role. Three roles need **three tokens**, not the two the contract
> has today — and it makes the `.preLine` split necessary rather than optional,
> since prose and exhibit transcript share one element and now want different
> faces.
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

- [ ] Three-face type contract, third token, `--fd-scale` re-derived
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
