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
| `case-file` | **Case File** | Manila folder holding *typed sheets*; redaction bars, paperclips, photographs | Cases. Prints beautifully, which the Case dossier needs anyway |
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

**Evidence photographs reveal their detail on hover** — the print is a print
until you lean in.

> **This needs a non-hover path, and it is not optional.** Hover does not
> exist on a tablet or an interactive whiteboard, which is where a good share
> of these students will meet it. Hover is the *enhancement*; tap-to-reveal, or
> simply showing the detail, is the behaviour. An affordance only reachable by
> mouse is the same class of failure as the `prefix` that was typed and never
> drawn — present in the design, absent for the child.

**Typeface: Special Elite**, a distressed typewriter face. Verified on Google
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
