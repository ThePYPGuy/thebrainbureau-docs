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

## Two rulings on pointer-only interactions

**This project has now turned down the same class of interaction three times,
and the reasoning is written in the code each time.** Recorded together because
the fourth request will arrive looking different again.

**HOVER: refused.** `Zoom.tsx:16` and `Tasks.tsx:323` — *`mouseenter` either
never fires or fires once on tap and then sticks* on the tablets and interactive
whiteboards these children use.

**DRAG IN A PACED WINDOW: refused.** `lib/banks/question-types.ts:9` — ordering
and matching question types are absent because *both need drag interactions, and
neither fits a projector-paced answer window.*

**DRAG AS A PRIMARY INPUT: refused, 2 Sep.** Tailwind's route sequencer is
specified as *drag-to-sequence*. **Build it as an ordering component that also
accepts drag, not a drag component with a fallback.** Tap a tile, tap a slot;
arrow keys move a focused tile; drag layered on afterwards and droppable if it
costs anything. Drag is imprecise across a large IWB, fights page scroll on a
tablet, and does not exist at all for a keyboard.

**The test is the repo's own sentence:** *the affordance that everybody has is
the one worth having.* And **a name in a spec is not an interaction decision** —
"drag-to-sequence" is what the component was called in a table whose stated
first user has never been built.

**None of these removes anything.** Drag can still be layered on; hover cannot,
which is why only one of the three is permanent.
