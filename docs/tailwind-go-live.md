# Getting Operation Tailwind live

> ## STOP — DO NOT RUN `npm run import -- --prod`
>
> **Maciej played it on 2 Sep and found four defects, three of them blocking.**
> Everything else in this file has been done: both branches merged, 569 tests
> green, typecheck clean. **Production is untouched — still at migration 34, and
> the Operation does not exist there because the import has not run.** That is
> the only thing standing between these defects and a classroom.
>
> 1. **Item 5, the daylight chart, has NO ROWS.** The task authors `rankValues`;
>    the `ordering` path reads `loadSelectionRows(config.selectionId ?? "countries")`
>    at `check.ts:198`, falls back to `countries`, and renders Global Intel
>    Cards' headers with nothing under them.
> 2. **Both matching items show the answer**, each left card printing the feature
>    and its function together. There is nothing to match.
> 3. **Item 10, the console, does not work** — six numbered text boxes rather than
>    an ordering control. The finale: exhaustively graded, never drawn.
> 4. **The room does not render.** Skin applied, scene absent.
>
> **Items 1, 6 and 7 — the wall map and both insets — work and are good.**
>
> Operation Builder is fixing them. **Each fix is proven by opening it in a
> browser, not by a passing test**, for the reason below.


> ## The grid-exact blocker is CLEARED — verified 2 Sep
>
> `tailwind` shipped a grader that **refused 75% of correct clicks on item 6**:
> `validate.ts` compared `round1()` while the crosshair readout the child looks
> at TRUNCATES, so a click at 41.75 to 41.79 displayed `417` above their finger
> and was graded 41.8. The task's own hint says *easting 41.7, northing 26.8*,
> so a child following it exactly had a one-in-four chance of being believed.
>
> **Fixed on `fix-grid-exact` at `060ccff`.** Verified here, not relayed:
>
>     round1(gx) in validate.ts                0
>     files vs tailwind                        point-space.ts, point.test.ts, validate.ts
>     anything from lib/locks/                 none
>     merges into main                         clean
>
> **The branch was empty for a window and I nearly cleared it too early.** If
> you ever need to re-check it, those first three lines are the test — a branch
> named for a fix merges just as cleanly when it carries nothing.
>
> Found by an ACCEPTANCE case and findable by nothing else: every refusal test
> passed before and after, because a grader one tenth out refuses wrong answers
> perfectly well. The item had been played in a browser and signed off.

---

## The order, and why it is this order

**MIGRATIONS BEFORE THE CODE PUSH.** This is a correction: an earlier version of
this file put the push first, on the grounds that 40 and 42 gate the IMPORT
rather than the app. That is true of 40 and 42 and **false of 41**.

**Migration 41 is `auth_attempts`, and the PIN rate limiter on `main` needs it at
RUN TIME.** `lib/server/rate-limit.ts` reads and upserts that table at three
sites, has four bare `catch {}` blocks, logs nothing, and says so itself at line
173: *never throws and never blocks on error*. **So deploying the code without
the table gives you a limiter that silently does nothing** — and nothing anywhere
would say so. Nothing regresses, because production has no limiter today either;
what changes is that everything written down about the protection becomes true
of the code and false of the system.

Found by Website Designer, verified here rather than relayed.

`supabase db push` sends migration FILES to production and does not need the
code deployed, so migrations-first costs nothing and closes that window.

**Both branches must merge before anything is pushed.** Production has none of
35..42 applied. Push `main` alone and it applies 35, 38, 39, 41; `tailwind`'s 36,
37, 40 and 42 would then arrive numbered BELOW migrations already run. Merge both
first and all eight apply in order, first time.

**Ignore the local database's state.** Its ledger, its schema and the files
disagree in all three directions, because four migrations were applied as raw
DDL to avoid rewriting shared history. **Production is the clean case.** The
local mess is not evidence of a deployment risk.

---

## 1. Merge both branches — ordinary merges, not fast-forwards

    cd ~/thebrainbureau && git merge --no-edit fix-grid-exact && git merge --no-edit tailwind-support && git log --oneline -1

**`fix-grid-exact` CONTAINS `tailwind`**, so merging it brings the Operation and
the grader fix together — you do not merge `tailwind` separately.

**An earlier version of this file said both were fast-forwards. That was true
when it was written and I invalidated it myself**, with 22 docs commits to
`main` afterwards. `--ff-only` would now fail on both. **A runbook that asserts
a merge is a fast-forward goes stale the moment anyone commits to `main`** —
including its author. What survives is that neither CONFLICTS, checked with
`git merge-tree --write-tree main <branch>` against `main` as it stands now.

Migrations after both land: 35 through 42, unbroken.

## 2. Restore the push remote

    git -C ~/thebrainbureau remote set-url --push origin https://github.com/ThePYPGuy/thebrainbureau.git && git -C ~/thebrainbureau remote -v

## 3. Apply the migrations FIRST

    cd ~/thebrainbureau && npx supabase db push

Expect 35 through 42, in order, none already applied. **41 is the one with a
deadline** — the PIN limiter is inert without it and says nothing.

## 4. Push — this deploys the code

    cd ~/thebrainbureau && git push origin main

## 5. Upload the five printables and the ZIP

Each writes production storage and will ask before it does. The ZIP is already
rebuilt with the corrected clue-card-pack; the five PDFs are the current ones.

    cd ~/thebrainbureau
    H="/mnt/c/Users/Admin/Documents/The Brain Bureau/Operations/Tailwind/Operation Tailwind  Build Handoff/printables"
    npm run upload:resource -- --prod operation-tailwind "$H/bureau-reference-shelf.pdf"     "Bureau Reference Shelf"
    npm run upload:resource -- --prod operation-tailwind "$H/agent-evidence-log.pdf"         "Agent Evidence Log"
    npm run upload:resource -- --prod operation-tailwind "$H/clue-card-pack.pdf"             "Clue Card Pack"
    npm run upload:resource -- --prod operation-tailwind "$H/teacher-answer-key.pdf"         "Teacher Answer Key"
    npm run upload:resource -- --prod operation-tailwind "$H/junior-agent-certificate.pdf"   "Junior Agent Certificate"
    npm run upload:resource -- --prod operation-tailwind "$H/operation-tailwind-printables.zip" "All printables (ZIP)"

**Note the two spaces** in `Operation Tailwind  Build Handoff`. **[verify]** the
slug is `operation-tailwind` and the label argument is wanted — `npm run
upload:resource` with no arguments prints its own usage.

## 6. Verify what actually landed

    cd ~/thebrainbureau && npm run upload:resource -- --verify --prod

This is the step that proves every listed path is really in the bucket. It reads
only.

## 7. Import the content — AFTER the migrations

    cd ~/thebrainbureau && npm run import -- --prod

This is the step the CHECK constraint gates, which is why it comes after 3.

## 8. Check the deployed state against the repo

    cd ~/thebrainbureau && npm run deploy:check -- --prod

---

## Then look at it yourself, and here is where to look

**Six of the nine items were never opened in a browser.** Items 1 and 9 were
played end to end. The rest are proven by unit tests and by the fixed-facts
script, not by use.

**The console has never been drawn.** All 480 combinations are enumerated and
graded — one accepted, 479 refused, one refusal shape — and the thing has not
appeared on a screen. Reaching it needs eight findings, which is why nobody has.

**A hard navigation to `/terminal` can hang on `Loading`** — a 200, nothing
logged. The click-through path from the assignments page works, which is the
route a child takes. Unexplained rather than fixed.

**And the twelve props are worth your eye rather than mine.** They were
generated against your own situation room, and the three console assets had to
be regenerated without a reference because at 16:9 the reference handed over the
composition and returned the room itself.
