# Getting Operation Tailwind live

> ## STOP — ONE BLOCKER, ADDED 2 SEP
>
> **`tailwind` carries a grader that marks correct answers wrong on item 6.**
> Do not run this sequence until the fix is on the branch being pushed.
>
> `validate.ts:486` compares `round1()`. The crosshair readout the child looks
> at TRUNCATES — `sixFigure` and `gridReading` are both `Math.floor`. So a click
> at 41.75 to 41.79 displays `417` above the child's finger and is graded 41.8.
>
> Measured against Tailwind's real answer, 10,000 clicks whose readout showed
> the target reference:
>
>     ACCEPTED by the shipping grader                 2,500
>     REFUSED though the readout said they were right 7,500   (75%)
>
> Only the lower-left quarter of the correct cell is accepted. **And the task's
> own hint says "easting 41.7, northing 26.8"** — so a child following the hint
> exactly has a one-in-four chance of being believed.
>
> Fixed at `0acba11` on `lock-library`, which cannot merge yet — it carries an
> unfinished library. The engine-only half goes onto `fix-grid-exact`, cut from
> `tailwind`, which then merges alongside it. Add that merge to step 1.
>
> ### THE BRANCH EXISTS AND IS EMPTY — DO NOT MERGE IT YET
>
> As of this writing `fix-grid-exact` is **identical to `tailwind`**:
>
>     git diff --name-only tailwind...fix-grid-exact   -> empty
>     git log --oneline tailwind..fix-grid-exact       -> empty
>     validate.ts:486                                  -> still round1(gx)
>
> **Merging it would succeed, change nothing, and ship the defect** — and it
> would succeed cleanly, because there is nothing in it to conflict. A branch
> named for a fix reads as a cleared blocker.
>
> **Verify before you merge it, with these two commands:**
>
>     git show fix-grid-exact:lib/engine/validate.ts | grep -c "round1(gx)"
>     git diff --name-only tailwind...fix-grid-exact
>
> **0 and three engine files** means the fix is really on it. Anything else
> means it is not, whatever the branch is called.
>
> **`main` is unaffected**: it does not have this validator at all. `tailwind` is
> the only branch carrying the defect into production.
>
> Found by an ACCEPTANCE case and findable by nothing else. Every refusal test
> passed before and after — a grader one tenth out refuses wrong answers
> perfectly well. **It surfaced because a reference instance asserted that a
> correct click must be accepted**, which is the contract clause the spec never
> had. The item was played in a browser and signed off with the bug in it.


**Written for Maciej. Every step here is one only he can do** — push is disabled
at the git level, and the uploads and the production import all write to
production and stop to ask by design.

**Run these from a plain WSL shell, not inside a Claude session.** Pasting a
command into a session makes the session ask permission for it, which is how the
first three attempts at this went sideways today.

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

## 1. Merge both branches — fast-forwards, verified clean

    cd ~/thebrainbureau && git merge --ff-only tailwind && git merge --ff-only tailwind-support && git log --oneline -1

`tailwind` is 0 behind `main` and 36 ahead. `tailwind-support` carries the
overview page, the credits component and no migrations, and touches no file that
`tailwind` touches. `git merge-tree` reported no conflicts for either.

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
