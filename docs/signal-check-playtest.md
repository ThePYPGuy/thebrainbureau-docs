# Signal Check — first play, 1 Sep 2026

Maciej played a live game as a guest, on a phone, joining through *Enter a code*.
**The first time anyone has played it.** It found more than every automated check
this project runs, which is the entry worth reading twice.

Ordered by what a classroom would suffer first.

---

## Broken

**A child is stranded when the teacher ends the game early.** The phone stayed on
the last question. **The page already handles this** — `app/live/play/[sessionId]`
renders a *Session complete* branch at `:198` — so the fault is that `ended` never
reached the device, not that nothing was built for it. Look at delivery, not at
the view. This is the one that happens in front of a class.

**The projector does not fit the screen.** It had to be zoomed out by hand. A
projector surface is the one display whose dimensions nobody can predict and
nobody can adjust mid-lesson; it has to fit whatever it is thrown at.

**The previous answer stays highlighted when the next question appears.** So a
child sees an option already chosen for a question they have not read.
`answeredPosition` is keyed by `question.position` and would reset on its own, so
whatever paints the highlight is probably held separately and not cleared.

**The highlight does not always appear when an option is chosen.** Unreliable
feedback is worse than none — a child who is not sure whether their tap landed
taps again, and the second tap is the one that surprises them.

---

## Confusing

**The join flow asks for an agent name, then asks again.** Entering a game PIN at
`/join` reveals codename and PIN fields, takes them, and *then* hands off to
`/live/join`, which asks for a display name. Two identity questions for one
arrival.

**This is a direct cost of a decision Doc Manager recommended.** `/join` reveals
its second step by counting six digits and deliberately does not resolve the code
first, so it cannot know a game PIN from a class code and shows the fields for the
commoner case. The reason was to stop the box becoming an oracle for which codes
exist.

**Maciej wants the code resolved first, then the right second step.** That is the
reversal, and the argument for it is now empirical rather than theoretical: the
oracle risk is small — a game PIN is explicitly *a room number on a whiteboard* —
and `/api/agent/exists` already resolves a code before answering, so the
principle is half-conceded already. **The class code is the sensitive one**, and
whatever is built should reveal less about that than about a PIN.

---

## Wrong by design

**Question text should be centred above the options**, on both surfaces.

**Options should be colour-coded** — blue, orange, black, white — on the projector
*and* the student device, so a teacher saying *"the orange one"* means something.
**Text colour is part of the requirement, not a detail:** black on blue and white
on orange are the pairs that fail, and this is a room reading at eight metres.

**Phone buttons are too small and stacked in a list.** They want to be a grid
sized to the screen — a thumb on a phone held one-handed, not a cursor.

---

## Missing

**End the question when everyone has answered.** Waiting out a countdown that
nothing can change is dead time in a lesson, thirty times over.

**Auto-advance after the reveal**, on a timer the teacher sets — three or five
seconds. Without it a teacher runs a twenty-question game by pressing a button
twenty times while also teaching.

---

## Not clear from the report

**Ending early showed a list of questions and how many answered each correctly.**
Whether that is the intended end screen or a fallback nobody designed is not
stated. Worth asking before anyone changes it.
