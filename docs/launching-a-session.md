# Launching a session

**Start Claude from a WSL terminal, inside the worktree. Never from the Claude
Desktop folder picker pointed at `\wsl.localhost\`.**

Two independent reasons, and each one alone is enough.

**It can't build.** `new-worktree.sh` says so at step 5 in its own words: a
session launched on Windows against the share *can read files and can run
neither npm nor git*. It survives by shelling out through `wsl -e bash -ic` for
every single command, which is slower on every command it runs.

**It can't be found.** A session's ADDRESS — the name other sessions use to
message it — is taken from where it CONNECTED, not from where it works and not
from its title. Launch it in the wrong folder and it is misnamed for its whole
life, whatever you rename it to afterwards.

Operation Builder is the worked example of both: launched in `tbb-quiz`, working
in `tbb-tailwind`, addressed `tbb-quiz-44`, and invisible to anyone searching
for a tailwind name. It cost six misidentifications and three stalls.

## One command, because two boxes is two chances to get it wrong

**Put the whole prompt inside `claude "..."` so there is nothing to paste
anywhere.** On 2026-09-02 the instructions said *run this, then paste that*, and
the prompt went into bash, which answered ``Command 'You' not found``. The shell
cannot tell that text was meant for a different program.

Chain with `&&` so a failure stops the run.

## A new worktree

    cd ~/thebrainbureau && scripts/new-worktree.sh --role 'Tailwind Support' support tailwind-support && cd ~/tbb-support && claude "You are Tailwind Support. You work only in ~/tbb-support, on branch tailwind-support. That folder is yours; no other worktree is. Read docs/local/briefs/tailwind-support.md -- it is written for you. Report by appending to docs/local/briefs/tailwind-support-progress.md and then starting the next piece of work in the same turn. Never end a turn on a report. Do not push -- Maciej pushes."

The script creates the worktree, copies `.env.local`, hard-links `node_modules`
and reports the next free migration number. It refuses if the worktree or the
branch already exists, which is why the `&&` matters — a refusal must stop the
chain rather than launch a session in the wrong place.

## A worktree that already exists

Drop the script; everything else is the same one command.

    cd ~/tbb-support && claude "You are Tailwind Support. You work only in ~/tbb-support, on branch tailwind-support. ..."

## What the prompt has to say

The ROLE, the FOLDER and the BRANCH; that commits land on that branch because
that is what the folder has checked out; where the brief is; **do not push**;
and the reporting rule — append to a progress file and start the next piece of
work in the same turn.

**The reporting line is not a formality.** A peer session ends its turn when it
reports and does not resume until something addresses it, so a report that needs
a reply is a stall waiting to happen. Appending to a file is not. And the rule
is really *do not stop while there is a next piece of work*: a blocker on one
item is not a blocker on the build.
