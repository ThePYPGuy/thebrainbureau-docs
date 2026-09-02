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

## A new worktree

    cd ~/thebrainbureau
    scripts/new-worktree.sh --role 'Operation Builder' tailwind tailwind

It creates the worktree, copies `.env.local`, hard-links `node_modules`, reports
the next free migration number, and prints the launch line and opening prompt.
Then, in a WSL terminal:

    cd ~/tbb-tailwind && claude

and paste the prompt it printed.

## A worktree that already exists

No script needed — it is the launch line and the prompt on their own:

    cd ~/tbb-tailwind && claude

Then paste, filling in the three names:

    You are <ROLE>. You work only in ~/tbb-<name>, on branch <branch>.
    That folder is yours; no other worktree is. Commits you make land on
    <branch> because that is what this folder has checked out.
    Read docs/local/briefs/<branch>.md -- it is written for you. If it is
    not there, ask rather than assume there is one.
    Report to Doc Manager by appending to
    docs/local/briefs/<branch>-progress.md and then starting the next piece
    of work in the same turn. Never end a turn on a report.
    Do not push -- Maciej pushes.

**The reporting line is not a formality.** A peer session ends its turn when it
reports and does not resume until something addresses it, so a report that needs
a reply is a stall waiting to happen. Appending to a file is not.
