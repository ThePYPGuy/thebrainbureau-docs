# Visual Identity and Skins

**Status: decided 2026-08-24.** The seven archetypes are locked, and the
Bureau face is light. `activities.skin` is a real, constrained column and
`npm run skins` reports on it; the CSS token scoping that lets a second skin
actually render is still to do.

---

## The two faces

The platform has two audiences with opposite needs, and one visual language
cannot serve both.

**The Bureau** — teacher-facing and institutional. Homepage, signup, dashboard,
library, reports, certificates, printed dossiers. This is where a Head of
Maths decides whether to trust you with a budget. It should be calm, legible,
credible and boring in the way official things are boring.

**The Field** — student-facing and immersive. The surface a child works on
during an activity. This should be varied, atmospheric, and never look like a
worksheet.

Your observation about Blooket, Kahoot, Gimkit, CIA and FBI all being white is
about the *first* face. Those are institutional surfaces, and institutions
signal trust by being plain and light. The dark, glowing terminal is not what
an intelligence agency looks like — it is what one looks like *in a film*.
Both are correct; they just belong on different screens.

### Current state — done

| Surface | Was | Now |
|---|---|---|
| Homepage, signup, dashboard | Dark navy `#0e1420` | Warm off-white `#f7f5f1`, ink text, gradient accents |
| Student terminal | CRT green/cyan on near-black | Unchanged |

Every text pair on the light surfaces was contrast-checked and clears WCAG AA,
the lowest being 4.86:1 on the amber used for join codes and the nav mark.

**The Bureau face is light.** A warm off-white, not a generic white SaaS look,
with the accent gradient kept for identity. The gradient keeps the bright
brand colours literally while the named colour tokens became ink-dark, because
the same variables are used for both decoration and text and a gradient built
from ink comes out muddy.

**The logo keeps its dark ground.** The mark was drawn for a dark banner and
reads as itself there, so on the light homepage it sits on a navy plate with a
sliver of the brand gradient along the bottom edge — deliberate, rather than a
hole punched in the page.

Why light wins for the Bureau face specifically:

- **Projectors and interactive whiteboards.** A dark UI projected into a lit
  classroom goes muddy. Kahoot is effectively a projector product and is white
  for that reason.
- **Printing.** Cases produce printable dossiers and the platform issues
  certificates. Dark surfaces either waste toner or need a separate print
  stylesheet every time — which the certificate already needed.
- **Extended reading.** Teachers read library listings, reports and rosters at
  length. Children read short bursts.
- **Procurement.** Schools read dark, neon software as a game. Light reads as a
  tool. You are selling to the person who signs the invoice, not the child.
- **Contrast is easier to guarantee** on light backgrounds, which matters for
  the accessibility commitments below.

The Field keeps dark because that is where being unlike Blooket is an asset
rather than a liability.

---

### The 2026-08-31 redesign replaces one face and not the other

Decided 31 Aug, and the line it follows is the one this section already draws.

**Replaced:** the **Bureau face** — the marketing site, the teacher dashboard and
the student dashboard. New identity, new palette, new typefaces: DM Serif
Display, Space Mono, Inter. `docs/brand.md` holds the specification.

**Untouched:** the **Field face** and all **seven archetypes**. The activity
interiors are locked, drawn work — Case File's typewriter and Field Terminal's
CRT are the product, not decoration. VT323, Special Elite and Courier Prime stay
exactly where they are.

**The trap is the root binding, and it is already recorded in `CLAUDE.md`.**
`app/layout.tsx` binds `--font-display` and `--font-body` on `<html>`, which sits
above every `[data-skin]` block — so replacing the fonts at the root reaches
every activity silently, and this project has already shipped a terminal
rendering a third of its text in a fallback face with nobody noticing. The new
faces must be **scoped to the Bureau surfaces**, not bound at the root, and the
old three must keep resolving inside `[data-skin]`.

Six families now load rather than three. That is a deliberate cost of running two
registers, and it is a reason to scope rather than a reason to compromise.

**No exception was needed, and one was nearly made.** The Signal Check projector
was commissioned on 2026-08-31 and first assigned to Website Designer, on the
reasoning that design should stay in one session. Maciej asked why the session
that built the screen was not simply finishing it, and the reasoning did not
survive the question.

**Design work is not one job here; it is two registers that share nothing.** A
Field Terminal projector is VT323 and CRT tokens. Website Designer's whole
context is DM Serif, Space Mono, navy and teal — a face that must never mix with
it. There was no consistency to preserve, so the boundary bought nothing and
cost an exception.

So a Field surface stays with the session that built it. That session also
already holds the things the work needs: the override mechanism and its
`projected` trigger, the live state machine, and a load harness that can put
thirty clients in front of it.

Kept here because the argument was plausible, and the next plausible one will
also arrive as a reason to move the fence.

### What the projector taught, built 2026-08-31

**Contrast is distance-dependent and the ratio is not.** Field Terminal's
`#d0d0d0` is 13:1 on its ground — ample at arm's length, thin at eight metres.
The projected variant goes to `#ffffff` on `#05070a`, 20.6:1. A figure that
passes AA says nothing about a wall.

**One accent, and it is not the brand one.** Magenta on black is 6.9:1 and goes
muddy through a lamp; cyan carries the chrome instead.

**A display face has a stroke width, and distance eats it.** VT323 stays on the
PIN, the timer and the counts, where the type is huge. It comes *off* the
question, because a one-pixel stroke blooms at distance.

**A shared screen cannot letter its options.** The obvious move is A/B/C on the
wall, and it is wrong here: options are shuffled **per player**, so the wall
shows the canonical order and every device shows a permutation. "B" on the wall
is a different option on every screen in the room. Any future mode with a shared
surface inherits this.

**Correctness is triple-coded** on the reveal — a tick, a full bar and a solid
edge. Remove the colour and it still reads, which is the accessibility rule
applied to a surface nobody can adjust from their seat.

The palettes differ by trigger; the shared list does not. No texture, motion off,
plain body face, correctness by shape and position — those hold for a person's
setting and for a projector alike, and that is the part that must not drift.

### Which surface is which

The line is what a child is doing, not who they are.

| Surface | Face | Why |
|---|---|---|
| Assignments, account, classes, results, store | **Bureau (light)** | Administration. Read at length, sometimes by a teacher over their shoulder. |
| The mission itself, Agent Training | **Field (dark, themed)** | The work. Where atmosphere earns its keep. |

Every door — deployment code, class code, front door — lands a student on
their **assignments page**, never straight inside a mission. A child sees what
they have been set before they drop into the field, and there is one place
they know how to get back to.

> A mission board was briefly built inside the CRT, which put "here is what
> you have been assigned" on the wrong side of this line. Corrected
> 2026-08-24: the assignments page absorbed it, and the CRT board was removed
> rather than left as a second way to do the same thing.

### `/join` stays on the Field face — decided 2026-08-31

Website Designer asked rather than assumed, and the answer is no change.

The table above lists *account* as Bureau, which makes a signing-in page look
like administration. It is not. **`/join` is the door, not the account.** A
child types a codename and arrives — the page `redirect`s to `/profile` — and
everything behind the door that is administration is Bureau. The door itself is
the moment of arriving somewhere, and the boot sequence is what says so.

Three things settled it. The Bureau face exists to explain the platform to a
teacher deciding whether to use it, and **a child is not that audience.** The
accessibility worry that would otherwise argue for moving a CRT page is already
answered elsewhere — clear view beats every skin and is remembered per student,
so no child is stuck reading a terminal they cannot read. And moving it would
have changed a written decision rather than fixed a defect.

**The two student doors are not uniform, and that is not yet deliberate.**
`/live/join` is `data-skin="field-terminal"` with no bezel and no boot sequence;
`/join` has both. Field either way, so nothing here is wrong — but if phase 2
wants one student entry style, that is the difference to settle, and it is a
different question from this one.

---

## What stays constant, what varies

Variety without a system is just inconsistency. A child should never have to
relearn how the platform works because the art changed.

**Constant — the Bureau's grammar:**

- Where agent identity sits: codename, Intel, clearance, always the same place
- How you answer, submit and get told you were right
- How progress is shown
- The typographic *scale* and spacing rhythm, even when the faces differ
- Bureau furniture: the seal, the word AGENT, clearance titles
- Feedback vocabulary — a granted/denied pair, whatever the words

**Variable — the mission's costume:**

- Palette
- Type faces, within the fixed scale
- Background texture and imagery
- Character art and artefacts
- The device metaphor — see below
- Sound, if it is ever added

The test: a child dropped into an unfamiliar Operation should know how to play
within five seconds, and still feel they have gone somewhere new.

---

## Skin archetypes — locked

The list is closed. A new archetype is a design decision that gets agreed and
added here first, and the database constraint is what forces that conversation
rather than letting a one-off slug slip in unnoticed.

| `skin` | Name | Feel | Suits |
|---|---|---|---|
| `field-terminal` | **Field Terminal** *(built)* | CRT green, scanlines, monospace | Hacking, codes, systems, Agent Training |
| `case-file` | **Case File** *(built to stage 2, in use)* | Manila folder holding *typed sheets*; redaction bars, paperclips, photographs | Cases. Prints beautifully, which the Case dossier needs anyway |
| `archive` | **The Archive** | Artefact cards, museum labels, glass, aged paper | Objects and characters, history, cross-curricular |
| `situation-room` | **Situation Room** | Wall maps, pins and string, briefing boards | Case openings, geography, anything spatial |
| `surveillance-feed` | **Surveillance Feed** | Camera grid, timestamps, overlays, thermal | Observation, data, statistics |
| `field-notebook` | **Field Notebook** | Handwriting, sketches, tape, pressed specimens | Field Ops, science, outdoors |
| `lab-bench` | **Lab Bench** | Specimen trays, instruments, clean light | Science investigations |

### Case File: paper on the folder, never on it

**Decided 2026-08-25.** Nothing is written directly on the manila. The folder
is the container; the reading matter sits on **sheets laid on it** — a typed
page with its own edge, shadow and slight rotation, as though someone put it
there. Prose, exhibit transcripts and lock text all belong on paper. The folder
carries only what a folder carries: the tab, the case number, the texture.

The distinction is the whole archetype. Text on manila reads as a website
wearing a colour; text on a sheet reads as a document someone handled.

**Clicking an exhibit opens its facsimile. Click only — not hover.** Corrected
2026-08-30; an earlier note here said hover, which was wrong. Hover does not
exist on a tablet or an interactive whiteboard, where a good share of these
students will meet this, and an affordance reachable only by mouse is the same
failure as the `prefix` that was typed and never drawn.

**The facsimile carries the exhibit's data, so the page does not repeat it.**
The lock states the task; the figures live in the drawn object. That keeps the
page uncluttered and the exhibit worth opening.

> **This holds only while the facsimile is real DOM text**, which is the whole
> reason it is drawn rather than magnified. Take the transcript off the page
> while the exhibit is still a bitmap and the figures become unreachable — to a
> screen reader, and to anyone who cannot resolve pixel art. **Draw the
> facsimiles first, remove the on-page transcript second.** In that order, or
> the activity is briefly unsolvable for the children least able to say so.

**Three faces, three roles.** Decided 2026-08-30.

| Role | Face | Used for |
|---|---|---|
| Printed | **Archivo Narrow** | tab, stamps, headings, lock names, case number |
| Typed | **Special Elite** | the briefing memo — what a typewriter produced |
| Read | **Lora** | lock prose and anything a child reads repeatedly |

The first split is diegetic: the folder's labels were printed before anyone
typed anything, so a single face for both loses the distinction and the surface
reads flat even when the paper is right.

The second is a legibility decision rather than an aesthetic one. Special Elite
is distressed and irregular, which is right for a memo and hard work across
paragraphs a Y6 reader returns to — particularly a dyslexic one. Lora is drawn
for reading. **Atmosphere where it is read once; legibility where it is read
often.**

All three verified on Google Fonts. Lora and Special Elite already load;
Archivo Narrow is the only addition.

> **Two consequences worth planning for.**
>
> `--fd-scale` is `.79` for this skin, measured. It was `.667`, exact because
> VT323 and Courier Prime are both monospace and share one ratio for any
> string. **A proportional display face has no single ratio** — Archivo Narrow
> measured 0.786 to 1.002 across six strings. `.79` is the floor of that
> range, which is a floor rather than an answer: too small only renders small,
> too large overflows. Explicit sizes are the real fix, and a platform change.
>
> Three roles need three tokens, not the two `--skin-font-display` and
> `--skin-font-body` the contract has today. And it makes the `.preLine`
> markup split **necessary rather than optional**: prose and exhibit transcript
> currently share one element, and they now want different faces.

Palette, from `docs/mockups/case-file-appearance.md`: paper `#d9c9a3` on frame
`#14171c`, ink `#3a2c1a`, stamp red `#8c2f1f`, stamp green `#2f5d3a`, amber
`#b3791a`. Lighter papers `#f3ead4` and `#faf6ea` sit on the folder for the
title box and the memo.

**Marks of handling carry the archetype**: a rotated ACTIVE CASE stamp, CLEARED
stamps on solved locks, a drawn paperclip on the memo, per-lock icons, ruled
paper. The earlier note about Special Elite alone: Verified on Google
Fonts, so it arrives through `next/font/google` beside the other four — no new
loading path, no font binaries in the repo, no licence to track.

TT2020 was the first choice and is the better face, having real per-glyph
irregularity. It is **not on Google Fonts** — verified, every variant of the
family returns 400 — so it would need its OFL files committed and
`next/font/local`. Worth revisiting if the sheets read as too clean, but the
paper treatment carries most of the effect and the face carries the rest.

> Whichever face is used, **verify it actually loaded** by rendered width
> against a deliberately nonexistent family, per `CLAUDE.md`. A name added to a
> token without the file behind it falls back and looks approximately right,
> which is how VT323 rendered a third oversized for four commits.

### Keeping them varied

`activities.skin` is a real column with a CHECK constraint on those seven
values, not a key buried in the `theme` blob, for one reason: it has to be
countable. The failure mode here is quiet — five Operations ship in a row
wearing the green terminal because it was the path of least resistance and
nobody was counting.

```bash
npm run skins
```

Reports how many activities wear each archetype, names the untried ones and
what they suit, and prompts explicitly when the last three activities share a
skin or when only one archetype has ever been used. **Run it before authoring
any new activity.** That instruction is also in `CLAUDE.md`, so it happens
without being remembered.

Operation 2 — characters and artefacts — reads as **The Archive** or **Case
File**. Either would sit a long way from the CRT without leaving the world.

Note that three of these are *light* surfaces. Paper, museum and lab are warm
and bright, and would give more contrast against the Field Terminal than
another dark theme ever could. Variety is not only about palette, but palette
is the fastest way a child perceives it.

---

## How skins should work

Both existing identities already use CSS custom properties — `brand.module.css`
scopes its tokens to `.shell`, and the terminal defines its tokens on `:root`.
That is most of the way there.

What is missing is **scoping**. Terminal tokens live on `:root`, so there can
only ever be one student skin. Moving them under a skin selector, and having
components read tokens rather than named colours, means a new skin is a token
set plus optional artwork — not a new stylesheet.

The activity file carries `theme: { skin, accent }`, the importer copies the
skin into its own column, and both shipped activities now say
`field-terminal` — renamed from the working title `crt-terminal` to match the
fiction, since the brand plate on the monitor has always read
"G.E.B. FIELD TERMINAL — MK.1". The authoring format and the data anticipated
this; the CSS has not caught up.

**Do this before the five Operations exist.** Five bespoke stylesheets is the
point at which it stops being refactorable, and each one can independently
fail contrast, so you would be auditing five designs instead of five token
sets.

### Where a live session's skin comes from — decided 2026-08-31

`activities.skin` is a column on **activities**, and a live Agent Training
session is a bank plus a mode, not an activity. So Signal Check appeared to have
nowhere to declare a look. The answer is that it does not need a column.

**A mode declares its skin in code.** `question-banks.md` already settles the
half that decides this: a game mode is *"the rules — code, not data"*. Its look
is part of it, so it is declared where the mode is declared.

The two data answers are both wrong, and wrong in ways worth naming:

- **Not on the bank.** A bank is subject matter, and the entire premise of the
  question bank refactor is that a bank built for one mode runs in any future
  mode without being rebuilt. A skin on the bank makes the questions carry one
  mode's appearance to every other mode.
- **Not on the session.** A session is *one playing*. Putting the skin there
  makes the look a per-session setting, so two runs of the same mode could look
  different and a teacher would be offered a choice nobody meant to give.

`activities.skin` therefore stays exactly as it is. It answers what an authored
Operation wears, which is a real question about authored content, and it was
never the platform's skin registry.

**`npm run skins` keeps its scope, and should say what it is.** It counts
authored activities, to catch several shipping in a row wearing the same look.
Modes are three things in code and do not proliferate, so they do not belong in
that count — but the report reads as an inventory of the platform, and a
diagnostic read outside the question it was built for has already cost this
project a near-deletion. One line of output, not a change of scope.

### The projector is a display context, not an eighth archetype

Agent Training wears **Field Terminal**, which the table above already assigns
it. A projected surface needs scanlines off — they alias — much higher contrast,
larger type, and correctness signalled by shape or position as well as colour.

That is a *variant of* Field Terminal, not a new archetype. The seven stay
seven, and the constraint that forces the conversation stays unbothered. A
projector is a rendering condition, the way print is.

**Build it as the accessibility override with a second trigger, not as a second
mechanism.** The list below — high contrast, plain typeface, no texture, motion
off — is very nearly the projector list already; the difference is only that one
is triggered by a person's setting and the other by the surface being projected.
Two mechanisms doing one job will drift, and the one that must not drift is the
accessibility one.

---

## The accessibility override

If skins vary, the escape hatch must be global and must win over every skin:
high contrast, plain readable typeface, no texture, motion off. A student
setting, remembered across sessions, honoured everywhere.

Green-on-black with a pixel font and red/green correctness feedback is a hard
read for dyslexic and colour-blind pupils. That is survivable as one theme
among several with an override available; it is not survivable as the only
way to use the platform, and it is the kind of thing a school asks about
during procurement.

---

## Artwork

Nobody plans this until there are two hundred files and no convention.

- **Provenance.** Every asset needs a known source and licence — commissioned,
  bought, or generated. Recorded per activity, because "where did this picture
  come from" is a question a school may eventually ask.
- **Budgets.** School wifi is the constraint. A per-activity ceiling for total
  image weight, and no single asset large enough to stall a lesson.
- **Alt text is content, not an afterthought.** If an artefact carries
  information a task depends on, its description is part of the activity and
  belongs in the activity file.
- **Naming and location.** `public/activities/<slug>/` keeps an Operation's art
  with the Operation and makes deleting one clean.
