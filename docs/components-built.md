# What is actually built

The Operations series bible's §4.1 is headed *Components already built (reuse,
don't rebuild)* and says **check here before specifying anything from scratch.**
Three of its seven rows are not built, and a fourth describes behaviour that does
not exist. A reuse table that is wrong does not merely mislead one reader —
**it manufactures handoffs written forward**, where a spec calls something reuse
because its first user was planned rather than shipped. Tailwind's handoff was
written that way and cost a day of audit to unpick.

This file records what a grep actually finds, so the next specification has
something true to check against. **It is a snapshot; the commands are the
authority.** Re-run them rather than trusting the table below.

## Verified 2 Sep

| The bible says | What is there |
|---|---|
| Hover-to-investigate + click-to-zoom | **Click only.** `components/terminal/Zoom.tsx:16` and `Tasks.tsx:323` each explain, independently, that hover was REMOVED: *`mouseenter` either never fires or fires once on tap and then sticks* on the tablets and interactive whiteboards these children use. No `onMouseEnter` survives anywhere in `components/terminal/`. **Do not rebuild it** |
| Filterable suspect/data table, greying out eliminated rows | **No component, and no elimination state at all.** The filter is inline JSX inside `SelectionTask` (`components/terminal/Tasks.tsx`), bound to `state.dataset`, placeholder hardcoded `"Search countries..."`, with two further unshared render sites. `eliminat*` appears in the whole app only in comments and in authoring data (`_evidence.eliminates`). `Dossier.tsx:7` is explicit: *ten rows a child reads across while eliminating* — **the elimination is the child's own work on paper, and the table does not respond to it** |
| Drag-to-sequence timeline, built for Encore | **Absent.** No Encore, no dnd-kit, no sortable, no reorder UI. Maciej: *"Encore is coming - the drag and drop thing will have to be built."* Note `lib/banks/question-types.ts:10` already rejects drag types for the live/projector context |
| Cover-integrity meter, built for Encore | **Absent.** A second Encore attribution; nothing in the app mentions it |
| Blank-prop + code-overlay | **Real.** `components/terminal/Facsimile.tsx`, 104 lines. Its header carries the rule worth keeping: a hotspot holds *readings, labels and blanks — the things printed on the object — and nothing about what any of it means* |

## The commands

    grep -rn "onMouseEnter" components/            # hover: expect nothing
    grep -rn "eliminat" --include=*.tsx components app lib
    grep -rli "coverIntegrity\|cover.integrity" components app lib
    grep -rn "drag\|sortable\|dnd" --include=*.tsx components/

**Two rows that are real are not two rows you can use unchanged.** `Zoom` and
`Facsimile` are built for the terminal's case-file chassis; anything else here
is an extraction job, and an extraction with three existing callers is a
refactor, not a reuse.
