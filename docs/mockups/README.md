# Mockups

A mockup is a **claim about the finished thing**. On this project every claim
has to be checkable, so a mockup lives here, in version control, next to a list
of what it claims.

## Why this folder exists

Operation Prime Directive was built without a Suspect Log or image zoom, both
of which a mockup had shown. Nobody had lost them — `git log --all -S "suspect"`
finds no such component in any commit. They were never here. The mockup was a
separate artefact, it looked finished, and nothing carried it across because
nothing was asked to.

It cost a full build-and-review cycle, and it surfaced only when someone played
the activity and could not solve it.

The same reasoning already moved the image style templates into
`content/styles/`: anything the platform's appearance depends on belongs where
the platform is.

## The rule

**One mockup, one capability list, both committed before building starts.**

```
docs/local/mockups/                   the mockup itself — NOT published
docs/mockups/<activity>.md            what it claims — published
```

The source lives under `docs/local/` because a mockup of an activity carries
its puzzle, and this folder is mirrored publicly. The capability list is safe
to publish and is the half other sessions read.

The `.md` is the load-bearing half. The file alone changes nothing — a
committed mockup nobody compares against is exactly as useless as an
uncommitted one. Write down every capability the mockup demonstrates, one line
each, in plain language a person can check by looking:

```markdown
- [ ] Suspect Log: ten suspects, portraits, six attribute columns
- [ ] Evidence images enlarge on click
- [ ] Briefing shown on load, typed on paper
- [ ] Case number on the folder tab
```

## How it connects to the rest

**Every unticked line is a `STATUS.md` §8 item** until it is built. That puts
the gap in the file every session reads, rather than in someone's memory of a
chat that has since scrolled away.

**An activity does not publish while its list has unticked lines.** That is the
gate. `designed: true` was believed to be that gate and never was; this is the
one that means something, because it is read by a person before they publish.

## What this does not fix

A mockup shows how something looks. It cannot show whether the puzzle works,
whether a ten-year-old can solve it, or whether the deduction lands. Prime
Directive's missing Suspect Log passed every automated check in this repo and
was found by playing.

So the list makes the *capabilities* checkable. **Playing it is still the only
thing that tests the experience**, and it has to be done by someone who does
not already know the answers.
