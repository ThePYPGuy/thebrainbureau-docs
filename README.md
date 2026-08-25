# Brain Bureau — documentation mirror

**Generated. Do not edit here.** This is a read-only mirror of the
documentation in the private platform repo, published so that project-
management conversations can read it. It is a *mirror*, not a second
source of truth — edit the private repo and run `npm run docs:sync`.

Anything committed here by hand is destroyed on the next sync.

Contains `STATUS.md`, `CLAUDE.md` and `docs/`, and nothing else: the sync
refuses to run if a file outside that allowlist would be copied.

`docs/local/` is held back — project refs, deployment names and paths on
Maciej's machine. None of it is a credential; it is simply of no use to a
reader here, and public history is permanent. If something below refers to
a path or an identifier you cannot see, that is where it lives.

## Links that will not work here

**`CLAUDE.md` opens with "See README.md first". It does not mean this
page.** It means the platform README — architecture, the two rules that
shape everything, and the recorded schema deviations — which is not
mirrored, because it describes how the engine works. This generated index
happens to occupy that filename, so the link resolves here instead of
breaking. There is nothing further up the path; you are not missing a
redirect.

For the same reason, links to `lib/`, `scripts/` and `docs/local/` go
nowhere here. They resolve in the private repo. `npm run docs:sync` lists
each one every time it runs.

