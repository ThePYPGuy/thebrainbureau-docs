# Getting Operation Tailwind live

> ## WHERE THIS STANDS — 2 Sep, late
>
> **Steps 1 to 6 are ready and worth running. STEP 7, THE IMPORT, IS NOT** — and
> that is now a design matter rather than a defect.
>
> **The Operation's board is the wrong shape.** It was specified as nine document
> hotspots pinned around a room — click one, zoom to the document, work it — plus
> five flavour hotspots. **It was built as a numbered list**, with the room added
> behind it as wallpaper. Maciej: *this is a terminal on top of the image of the
> room.* `docs/tailwind-acceptance.md` has the spec quote and the three loose ends
> it explains. **The import is the step that puts Tailwind in front of children,
> so it waits for the board.**
>
> **Everything else should still go.** Steps 1 to 6 ship the platform work, the
> Students and Builder pages, the PIN limiter with its table, and two verified
> grader fixes — none of which depends on Tailwind appearing.
>
> ### The two grader fixes, both verified here rather than relayed
>
> **GRID-EXACT — CLOSED, already on `main`.** `tailwind` shipped a grader comparing
> `round1()` while the crosshair readout TRUNCATES, so **7,500 of 10,000 clicks
> whose readout showed the target reference were refused** — and the task's own
> hint gave that reference, so a child following it exactly had a one-in-four
> chance of being believed. `main` now grades by `gridReading`. Confirmed twice.
>
> **CODE ORACLE — fixed on `fix-code-oracle`, ready to merge.** `offBy` returned a
> percentage on a wrong keypad code, so a guess plus its percentage narrowed a
> 10,000-way code to a handful in three or four submissions, with no lockouts to
> slow it. Verified: three engine files, one commit, nothing from `lib/locks`,
> `withoutDistance` present on the branch and absent from `main`, merges clean.
> **`operation-zero-hour` is PUBLISHED, so this one is live today.**
>
> Found by an ACCEPTANCE case and by nothing else: every refusal test passed
> before and after. `docs/answer-integrity.md`.

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

    cd ~/thebrainbureau && git merge --no-edit fix-code-oracle && git merge --no-edit tailwind-support && git log --oneline -1

**`tailwind` and `fix-grid-exact` are ALREADY MERGED into `main`** — done 2 Sep
and verified: `main` grades `grid-exact` by truncation. So the only code left to
bring is `fix-code-oracle` and `tailwind-support`.

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
