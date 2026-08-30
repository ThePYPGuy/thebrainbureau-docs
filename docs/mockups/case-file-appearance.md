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
> had.
>
> **No `.preLine` split is needed** — an earlier note here said otherwise. The
> transcript came off the page once the facsimiles carried the figures as text,
> so `.preLine` is prose only and takes the reading face. The job deleted
> itself, in the order it had to happen.
>
> **`--fd-scale` is `.79`, measured — and the mechanism is wrong.**
>
> The old `.667` was *exact*, because VT323 and Courier Prime are both
> monospace: their width ratio is identical for any string. Archivo Narrow is
> proportional, and the same six strings gave **0.786 to 1.002 — a 28%
> spread.** `.79` is the floor of that range, chosen because too small only
> renders small, while too large overflows.
>
> **A single scalar cannot describe a proportional face.** Replacing it with
> explicit sizes is a platform change — seventeen declarations shared with
> Field Terminal — and sits in §8.

## Palette

```
--paper       #d9c9a3     --ink         #3a2c1a
--paper-dark  #c9b688     --ink-soft    #6b5a3e
--frame       #14171c
```

**The mockup's `--stamp-red`, `--stamp-green` and `--amber` do not exist in the
build, deliberately.** The implementation reuses tokens the skin already had
rather than adding near-duplicates: `--accent-alt #7d2c25` for the stamp red,
`--ok #3f5c36` for the green, `--warn #6d4711` where the mockup says amber.
Grepping the stylesheet for `--stamp-red` finds nothing and proves nothing.

Two lighter papers sit on the folder: `#f3ead4` for the title box, `#faf6ea`
for the memo. Both carry ruled lines — a `repeating-linear-gradient` at 3px on
the folder, 22px on the memo.

Contrast must be measured, not assumed. `--ink-soft` on `--paper` is the pair
to check first, and `--amber` is the one that failed before at 3.54:1.

## Capability list

- [x] Three-face type contract, third token, `--fd-scale` re-derived
- [x] Folder: ruled paper, tab offset from the left edge, deep shadow
- [x] **ACTIVE CASE** stamp, rotated, translucent, top right — and **CASE CLOSED** once the file carries a completion box
- [x] Case progress bar — "6 OF 7 LOCKS CRACKED", green fill; both numbers counted in CSS off the list, not passed in
- [x] Title box on lighter paper; case number in the stamp red
- [x] Briefing as a **memo on its own sheet**, rotated ~-0.6°, ruled, with a
      drawn **paperclip**
- [ ] Lock rows: per-lock **SVG icon**, number, name — number and name done;
      **the icon is blocked.** Keying it to `:nth-child()` would put a lightning
      bolt on lock 1 of every Case File activity ever written, so it needs a
      field on the phase. §8
- [x] **CLEARED** stamp per solved lock, green, rotated ~-4°
- [x] **NEXT →** tag on the active lock, amber, outlined — and the row tinted
      **lighter** than the cleared rows, not amber-washed. An amber wash put
      NEXT at 4.44:1, and the open lock is the one row a child must be able to
      read. Written out so it does not get "corrected" back

Unticked lines are `STATUS.md` §8 items. The activity does not publish while
any remain.

## Not covered by this mockup

The lock page, the Suspect Log table, and the exhibit facsimiles — those are in
`operation-prime-directive.md`. This one is the index only.
