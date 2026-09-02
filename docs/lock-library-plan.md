# Lock Library — build plan

**Goal:** an Operation becomes a config file. Room choice + a lock-instance file
+ copy + blank props, and the engine renders the board, the validation, the
Evidence Log boxes and the answer key from it.

This plan assumes the spec and my review of it (`docs/lock-library-review.md`).
It answers the question the spec does not: **how it gets built without needing
Maciej in the loop.**

---

## 1. The operating model comes first, because that is what failed

Two sessions ran in parallel on Tailwind. One worked; one was closed for not
working independently. **The difference was not their competence.** Both wrote
good code and both caught real defects.

- **Operation Builder was ADDRESSABLE.** Doc Manager could message it, so a
  parked decision was answered in a minute. It stalled three times and each
  stall cost minutes.
- **Tailwind Support was NOT.** It appeared in nobody's `ListAgents` and could
  see nobody in its own, so every question routed through Maciej by copy-paste.
  Its defaults were right every time and it still could not proceed alone.

**A session that cannot be reached cannot be autonomous, however good it is.**
That is the lesson, and the plan is built around it.

### The rule: one addressable session, and subagents for the fan-out

**A peer session ends its turn and waits. A subagent runs to completion.** That
single difference is why twenty lock types should be built by subagents rather
than by a second and third peer session.

- **One builder session** owns the worktree, the shared engine, and the merge.
- **Per-type work fans out to subagents**, one type each, each owning its own
  files (`lib/locks/<type>.ts`, its test, its reference config). A subagent
  cannot stall waiting for a reply, and a type is a naturally file-isolated unit,
  so twenty of them do not collide.
- **The shared surfaces are built once, first, by the session** — never by a
  subagent, because that is where collisions happen.

### Non-negotiables carried from Tailwind

- Launch from a WSL terminal **inside** the worktree, so the session's address
  matches its work (`docs/launching-a-session.md`).
- An inbox file at `/home/maciej/tbb-inbox/<name>.md`, read before each new
  piece of work.
- A progress log appended to, never a report that ends a turn.
- **Do not stop while there is a next piece of work.** A blocker on one item is
  not a blocker on the build.

---

## 2. The contract, amended

The spec's six parts, plus the amendments the Tailwind build earned. **A type is
not done until all of these exist.**

1. **Config schema** — the JSON that fully defines one instance.
2. **Validation in the answer's native units** — grid units, degrees, order, set
   membership. **Never screen pixels.**
3. ~~Both delivery modes~~ — **teacher-panel entry is OUT of wave one, decided
   2 Sep.** Solo only. Strike the teacher-entry line from the type definitions
   or mark the whole column *wave two*, so nothing reads as built that is not.
4. **Evidence Log box pattern** — one of the six printed patterns.
5. **Answer-key row generator** — the key is assembled, never hand-written.
6. **Finding output** — every solved instance posts a named finding.
7. **NEW — proven by acceptance and refusal, in a browser.** At least one input
   that MUST be accepted and one that must be refused, and the type drawn and
   graded through a real click. *Nothing in the spec required a validator to
   accept anything*, and four separate guards passed this year by refusing
   everything. `matching` was unit-proven, guard-proven and never drawn.
8. **NEW — persistence proven by a round trip.** Solve, reload, still solved.
   Migration 37 shipped two tables with indexes and RLS and no writer; a lock
   that renders and grades looks perfect until a child logs out.

---

## 3. Wave 0 — foundations, one session, no subagents

**Nothing else starts until this is green.** These are the shared surfaces, and
sharing is what makes parallel work collide.

1. **The audit.** Survey what already exists against the twenty contracts.
   **Default assumption: a claim that something is built is FALSE until a grep
   names the file and a browser draws it.** Three planning documents have made
   that claim wrongly — the handoff (four claimed, one real), the bible's §4.1
   (three of seven wrong), and the spec itself (`drag-sequence` cites an Encore
   that does not exist; `table-lookup` is inline JSX, not a component). Record
   the audit at the top of the first PR.
2. **`LockInstance`** — the row shape, the config discriminated union, and the
   loader.
3. **The engine wrapper.** **Follow `tailwind:lib/engine/unlock.ts` rather than
   invent a shape** — 76 lines plus an 84-line test, named in none of the four
   planning documents and found by Lock Library's audit. It is already what
   this item asks for: **one pure evaluator, two thin callers**, on the
   principle that when the drawing and the enforcing disagree the gating model
   is decoration. **Take its refusal default with it** — an unrecognised rule
   keeps the phase SHUT, so an unparseable lock config must refuse rather than
   accept. The wrapper OWNS the series-wide behaviours: no lockouts,
   no partial feedback, one hint phrased as method. **Make these the wrapper's
   job, not each type's.** Twenty independently written types will diverge.
4. **Import as REPLACE, not merge.** Delete the Operation's instances and insert
   the current set, in one transaction. **Prove it by REMOVING an instance and
   re-importing** — a join table has no `on conflict` clause to forget, so it
   adds and never deletes, silently, while `content_hash` still moves and
   `deploy:check` reports agreement.
5. **`npm run locks`** — usage across Operations, and **flag ZERO usage**. A type
   nobody uses is a type nobody has tested in anger. Scope it to what is
   AUTHORED, not what is declared: `npm run skins` reported `situation-room` as
   worn while the skin had no CSS at all.
6. **The registry and ONE reference instance rendering.** **Decided 2 Sep: there
   is ONE page, teacher-facing, with the dev extras behind a flag** — not a
   private `/dev/locks` beside a public list. Two surfaces showing the same
   twenty locks is another pair that must agree.

   **The page belongs to Website Designer; the registry and the lock rendering
   belong here.** It lives under `/dashboard/builder`, reached from a Builder
   page offering *build your own case* and *build your own operation*. WD states
   the fields it needs as the consumer; this library implements exactly those.
   **The page renders the registry and never a hand-written list** — a list of
   twenty descriptions is a claim about what is implemented, and today the
   honest answer is five.

   **THE DEV FLAG MUST NOT BECOME AN ANSWER PATH.** `loadLockInstances` and
   `loadLockAnswer` read different tables and neither select list mentions the
   other, so a caller wanting the public half cannot receive an answer even by
   accident. **A `?dev` parameter is a view, never an authorisation** — adding an
   answer-carrying query to a teacher-facing route would undo the split that is
   the whole reason the answer has its own table. Showing a reference instance's
   answer is harmless in itself; reaching it through the public loader is not.

**Done when:** the audit is written, the wrapper enforces the three behaviours,
a removed instance disappears on re-import, and the registry renders one
reference instance inside Website Designer's page.

---

## 4. Wave 1 — promote the five that Tailwind already proves

**These are not new code.** They are working, browser-proven implementations
that get wrapped in the contract. Do them first because the audit will find
something real, and because there is a working Operation to check against.

| Type | What exists | Proven how |
|---|---|---|
| `pin-drop-map` | Tailwind item 1 | Real click; `READING 41°N, 12°W`; *Position confirmed* |
| `readout-map` | items 6/7, both inset maps | Extracted byte-for-byte from the pack source, integrity-asserted |
| `matching-pairs` | items 2, 8 | Drawn and graded in a browser |
| `verdict-select` | item 4 | Drawn and graded |
| `composite-finale` | the dish console | **480 enumerated: 1 accepted, 479 refused, one refusal shape** |

**Wave 1's definition of done is the spec's own, and it is the strongest test
available: Tailwind's items re-expressed as library configs with NO behaviour
change.** Same answers, same refusals, same findings, same Evidence Log. If a
re-expressed Tailwind plays identically, the abstraction is real. If it does
not, the abstraction is wrong and better found now than at type fourteen.

**Carry the console's lesson into `composite-finale` explicitly:** it must be
ONE submission. Authored as two tasks it fails by construction — two tasks
report separately, splitting a 480-way answer into a 120-way and a 4-way guess
and telling the child which half was wrong, **and both halves pass their own
tests.** And compute the answer space by ENUMERATING the accepted set, never by
multiplying declared cardinalities: `locked_slots` alone breaks the
multiplication, and a guard derived from the config it guards is not a second
guard.

---

## 5. Wave 2 — the seven with a near neighbour

Real code exists nearby; none of it is a library type yet.

`text-entry` · `keypad-code` · `table-lookup` (inline JSX in `Tasks.tsx`) ·
`drag-sequence` (**`OrderingTask` gives you the DATA path and none of the
interaction** — it renders numeric rank boxes at `Tasks.tsx:512`, not drag and
not tap-to-reorder, so the mechanic is new work. Corrected 2 Sep) · `rank-order` · `sort-bins` · `branching-key`

One subagent per type. Each ships its reference instance, which is what the
teacher-facing page renders for that type.

---

## 6. Wave 3 — the eight that are genuinely new

`hotspot-find` · `cipher-decode` · `dial-select` · `pattern-lock` ·
`image-assemble` · `progressive-reveal` · `spot-difference` · `gauge-set`

**These are new code wearing a familiar name.** Schedule them as new code.
`progressive-reveal` is a wrapper rather than an input and should be built last,
because it delegates to the others.

---

## 7. The rooms track, in parallel

The library binds to `canvas`, `rail` and `station` from each room's
`scene-layout.json`. **Sixteen backgrounds exist and exactly one is measured** —
SR-1, the Tailwind chassis. Fifteen to go, and measuring is the manual step that
becomes the new bottleneck.

- **Two aspect ratios of one room is two rooms.** Green Archive ships portrait
  and square; Red Archive landscape and portrait. Measure once in a normalised
  space and derive, the way the inset maps are re-derived from one source —
  otherwise the pair drifts, and drifts invisibly, because a region twenty
  pixels out looks like an untidy composite rather than an error.
- **Every room's layout gets a visual-regression baseline.** The harness already
  exists at `tests/visual/baseline/`. A wrong region is a visual fault and only
  a visual check will catch it.
- `store` and `flavour` have no lock binding. Either bind them or state plainly
  that Reference Files are placed by the room and never by a lock.

**Wave 1 needs only SR-1.** Do not block on the other fifteen.

---

## 8. The asset question, before there are fifty

The sixteen backgrounds are 3–4 MB PNGs; Tailwind's own food-chain photos are
1.5 MB and 1 MB for images that render circle-cropped and small. **That is a
real cost on a school connection.** They are painterly renders, so SVG is not
the answer — AVIF or WebP with responsive sizes is. Settle it once, in wave 0,
and every Operation after inherits it.

---

## 9. What would make me stop and re-plan

- **Wave 1 cannot re-express Tailwind without behaviour changes.** That means
  the contract is wrong, and finding out at type five is cheap.
- **The wrapper cannot own the three series behaviours** without each type
  knowing about them. That means the abstraction leaks and should be redrawn
  before nineteen types depend on it.
- **A type needs a room region that no room measures.** Then the rooms track is
  on the critical path after all, and its ordering changes.
