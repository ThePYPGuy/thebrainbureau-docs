# Getting Operation Tailwind live

**Written for Maciej. Every step here is one only he can do** — push is disabled
at the git level, and the uploads and the production import all write to
production and stop to ask by design.

**Run these from a plain WSL shell, not inside a Claude session.** Pasting a
command into a session makes the session ask permission for it, which is how the
first three attempts at this went sideways today.

---

## The order, and why it is this order

**Migrations must precede the CONTENT IMPORT, not the deploy.** Migrations 40
and 42 widen a CHECK constraint that gates `import`, not the app — nothing the
deployed code does sends a task type to the database. So pushing code before the
migrations land is harmless: the new renderers simply have nothing to draw until
content arrives.

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

## 3. Push — this deploys the code

    cd ~/thebrainbureau && git push origin main

## 4. Apply the migrations

    cd ~/thebrainbureau && npx supabase db push

Expect 35 through 42, in order, none already applied.

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

This is the step the CHECK constraint gates, which is why it comes after 4.

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
