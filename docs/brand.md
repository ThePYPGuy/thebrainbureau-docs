# Brand and Visual Identity

**Status: adopted 2026-08-31.** The identity for the Bureau face — the marketing
site, the teacher dashboard and the student dashboard.

**This document is the identity only.** It does not carry product scope, the data
model, the Intel economy or the activity schema; those live in
[`product-overview.md`](product-overview.md),
[`identity-and-access.md`](identity-and-access.md),
[`intel-and-clearance.md`](intel-and-clearance.md) and
[`activity-schema-v0.4.md`](activity-schema-v0.4.md). Layout specifications for
particular screens are build briefs with a shelf life and live in
`docs/local/briefs/`. Kept apart deliberately: a value repeated in two documents
is a value that will disagree with itself.

**Tagline:** Every Mission Is A Puzzle

**Audience:** primary teachers and their classes — initially Years 3–6 / Grades
3–5. The platform is not age-restricted, so **no age ranges in marketing copy.**

---

## Two registers, never mixed

| Register | Where | Feel |
|---|---|---|
| **Bureau face** | Marketing site, teacher dashboard, student dashboard | Light, clean, professional. Navy and gold. Speaks to adults. |
| **Field face** | Inside an activity | Dark, themed, atmospheric. Where the work happens. |

The homepage shows screenshots of the Field face and is itself always Bureau.
**Never apply activity styling to a Bureau surface, or Bureau styling to an
activity.**

[`visual-identity.md`](visual-identity.md) draws the line and holds the **seven
locked archetypes** the Field face is built from. This redesign replaces the
Bureau face and touches none of them.

---

## Typography

| Role | Font | Usage |
|---|---|---|
| Wordmark | DM Serif Display | "The Brain Bureau", nav and hero only |
| Headings | DM Serif Display | h1, h2 |
| Eyebrow labels | Space Mono | All-caps: `— The Operation`, `MISSION-BASED LEARNING` |
| Body | Inter | Body copy, nav links, descriptions |
| Buttons | Space Mono | All-caps, letter-spacing 0.08–0.14em |

Loaded through `next/font/google`, never `@import`.

**Scope them, do not bind them at the root.** `app/layout.tsx` binds
`--font-display` and `--font-body` on `<html>`, which sits *above* every
`[data-skin]` block — so a root-level swap reaches every activity silently. This
project has already shipped a terminal rendering a third of its text in a
fallback face, found by eye after three programmatic checks reported fine. The
activity faces — VT323, Special Elite, Courier Prime — must keep resolving inside
`[data-skin]`.

Six families now load rather than three. That is the price of running two
registers honestly.

---

## Colour

| Name | Hex | Usage |
|---|---|---|
| Navy | `#14284A` | Primary background — nav, hero, CTA strip |
| Dark navy | `#0A1628` | Footer, utility bar text, button text on teal |
| Mid navy | `#1A2035` | Dark section background |
| Teal | `#00C9A7` | Primary CTAs, active states, links |
| Blue | `#1D6EBF` | "Join a game" only — the student entry point |
| Gold | `#C4A35A` | Eyebrow labels, decorative arrows, seal accent |
| Cream | `#f5f4f0` | Light section background |
| White | `#ffffff` | Utility bar, cards |
| Body text | `#5A5550` | Body copy on light |
| Muted text | `rgba(255,255,255,0.55)` | Body copy on dark |
| Warm off-white | `#F2EEE4` | Headings and body on navy |

**Blue is reserved.** `#1D6EBF` marks the student's way in and nothing else. A
teacher scanning the page should be able to find the one button that is not for
them.

---

## Buttons

Exact values. Do not substitute Tailwind or component-library defaults.

All buttons: `font-family: Space Mono`, `text-transform: uppercase`,
`font-weight: 700`.

| Button | Background | Text | Border | Size | Padding | Radius |
|---|---|---|---|---|---|---|
| **Join a game** (utility bar) | `#1D6EBF` | `#ffffff` | none | 10px / 0.1em | 7px 16px | 3px |
| **Log in** (nav) | transparent | `#00C9A7` | 1.5px `#00C9A7` | 10px / 0.1em | 8px 18px | 3px |
| **Sign up** (nav) | `#00C9A7` | `#0A1628` | 1.5px `#00C9A7` | 10px / 0.1em | 8px 18px | 3px |
| **Primary CTA** (hero, CTA strip) | `#00C9A7` | `#0A1628` | 1.5px `#00C9A7` | 13px / 0.08em | 13px 28px | 4px |
| **Secondary CTA** | transparent | `#F2EEE4` | 1.5px `rgba(255,255,255,0.4)` | 13px / 0.08em | 13px 28px | 4px |
| **Section button** | `#00C9A7` | `#0A1628` | 1.5px `#00C9A7` | 11px / 0.08em | 11px 22px | 4px |

Size column is `font-size / letter-spacing`. The secondary CTA is the only one
without `font-weight: 700`.

---

## Type scale

| Element | Font | Size | Colour |
|---|---|---|---|
| H1 (hero) | DM Serif Display | 42px / 1.2 | `#F2EEE4` |
| H2 (section) | DM Serif Display | 32px / 1.2 | `#14284A` light · `#F2EEE4` dark |
| Section eyebrow | Space Mono | 16px / 0.18em, 700 | `#C4A35A` light · `#00C9A7` dark |
| Hero eyebrow | Space Mono | 10px / 0.22em | `#C4A35A` |
| Body | Inter | 14px / 1.8 | `#5A5550` light · `rgba(255,255,255,0.6)` dark |

**Section eyebrows are headers, not captions.** 16px and bold — they carry the
section, formatted `— The Operation` / `— The Case` / `— Agent Training`.

---

## Logo

Bronze bas-relief seal: navy ground, gold rope border, profile face with a
geometric network. Lives at `public/images/brain-bureau-seal.png`.

- Nav at 40px, plus hero, printed dossiers, certificates, favicon
- **Never in the nav without the wordmark** beside it in DM Serif Display
- Favicon is the seal alone
- The old circular badge with the yellow serrated border is **retired**

**Do not substitute a placeholder.** If the asset is missing, stop and ask for
it. A circular grey stand-in ships and then stays.

---

## Voice

Direct, warm, specific. Speaks to a teacher who has twenty minutes to decide
whether this is worth their Tuesday.

Say what the thing does. Avoid *engaging*, *transform*, *empower*, *gamified*.

**Signal Check** — the live mode, and the only Agent Training mode the site shows
while Mainframe Breach is unbuilt:

> Build a quiz in minutes, or choose from the Bureau Library, launch it live, and
> the whole class plays together on whatever device they have. Typed answers, not
> just multiple choice — so they work it out rather than pick it. No setup, no
> accounts. Just read the PIN and go!

The typed-answers clause is the differentiator: every competitor is four coloured
buttons, and a child typing `282` has done the multiplication.

**Do not market what is not built.** No tokens, no attacks, no AI-assisted
authoring, no leaderboard — the first three are unbuilt and the fourth never
will be.

---

## Layout reference

**Blooket**, for density and clarity rather than for looks. What is being taken:

- A narrow, quiet sidebar carrying one primary action and a short nav
- A **bank page that is the destination**, with the modes as buttons on it —
  Signal Check, Solo Practice, assign to a class. See
  [`question-banks.md`](question-banks.md).
- Question lists that reflow from one column to two as width allows
- Creation and editing screens with the same clarity as the reading screens —
  a bank editor is a place teachers spend real time and it is not a settings page

The brand is ours; the arrangement is the reference.

---

## Assets

| Asset | Path | Status |
|---|---|---|
| Seal | `public/images/brain-bureau-seal.png` | supplied 2026-08-31 |
| Field face screenshots | — | Case File and Field Terminal, for homepage cards |

Card previews on the homepage use **real screenshots**, not coloured
placeholders. Both faces exist and can be captured.
