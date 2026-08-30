# The activity file

**Partial.** This covers the fields an author has to know about and cannot
discover, starting with the ones that carry an Operation's voice. It is not a
schema — `activity-schema-v0.4.md` is referred to in places and has never been
written, and `README.md` records the deviations from it.

Every field here was read out of the code that consumes it, and the file names
that code so the next person can check rather than trust this page.

## `theme` — the Operation's voice

```json
"theme": {
  "skin": "field-terminal",
  "accent": "#FF5555",
  "timer": { "minutes": 45, "expiredLabel": "⚠ ZERO HOUR", "doneLabel": "VAULT SECURE" },
  "code":  { "label": "RESTORE CODE" }
}
```

| Field | Default if absent | Read at |
|---|---|---|
| `timer.minutes` | no timer at all | `Mission.tsx:180` |
| `timer.expiredLabel` | `⏱ TIME UP` | `Mission.tsx:182` |
| `timer.doneLabel` | `COMPLETE` | `Mission.tsx:183` |
| `code.label` | `CODE` | `Mission.tsx:163` |

**The defaults are deliberately voiceless.** They were `⚠ ZERO HOUR` and
`VAULT SECURE` until `c11d11e` — one Operation's fiction shown to every
activity that had a clock, so a workshop short-circuit case ended by securing
a vault nobody had mentioned. An author who wants a voice now has to write one,
and an author who does not gets words that fit any case.

`timer.hard` is recorded against a competitive mode that does not exist. Every
timer is soft: the clock striking changes the clock, never the mission.

## Fields that exist and are documented where they are consumed

Named here so an author knows to go looking, rather than described twice:

| Field | What it is | Where |
|---|---|---|
| `phases[].icon` | one of a named set, validated at import | `lib/phase-icons.ts` |
| `config.inputs.fields[].prefix` | drawn before the input — `$`, `C-0` | `Tasks.tsx` |
| `config.evidence` | **public**: `image`, `alt`, `caption` | `docs/mockups/` |
| `config._evidenceDesign` | **private**: eliminations, columns, traps | stripped at import |
| `orders.dossier` | the Suspect Log — columns and suspects | `Dossier.tsx` |
| `completion` | released only once every phase is done | `state.ts` |

## The rule that governs all of it

**Underscore means never public. Everything else is public, and `completion` is
public late.** A `_`-prefixed key is stripped on the way in and on the way out;
anything else you write reaches the browser on first load.

That is a rule about *where* a value sits, and it cannot see what a value
*says*. `config.evidence.caption` once read "C-09 COGSWORTH" — a legitimately
public field carrying Lock 07's answer, in an activity whose boundary was drawn
correctly. `npm run test:answer-leak` now checks values as well as shape, with
the floors stated in `STATUS.md` §10.
