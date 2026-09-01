# Brain Bureau — Activity Schema v0.4

**Purpose.** An *activity* is data, not code. The app is an engine that renders any activity conforming to this schema. Operations, Cases, Agent Training and Field Ops are the same shape with different settings.

This document is the contract. If a new activity needs an engine change, the schema is wrong — fix the schema, don't special-case the engine.

**Changed in v0.4**
- `briefing` renamed to `operation` throughout — cleaner, more Bureau-native, no collision with the platform's investigative framing
- Agent Training added as a fourth activity type (`training`) with `QuestionBank` as a separate object
- Activity types table updated

---

## 1. Design rules

1. **Answer keys never reach the browser.** Validation is server-side. Students will open devtools; assume it.
2. **Every task carries a curriculum tag.** Cheap now, impossible to retrofit across a library.
3. **Offline tasks are first-class.** Paper working is the design, not a gap in it.
4. **No task depends on reaction speed.** No timers, no speed bonuses. Paper-first working must never be penalised.
5. **Text is content, not code.** All student-facing wording lives in the activity file.
6. **Student records hold nothing identifying by default.** Codename plus PIN. Student SSO is opt-in per school, never required to run an activity.
7. **A teacher's availability never blocks a student.** Anything requiring an adult is off the critical path.

---

## 2. Activity types

| type | shape | live/async | duration | has documents | phase |
|---|---|---|---|---|---|
| `operation` | one phase, sequential puzzles | async | one lesson | rarely | 1 |
| `case` | multiple phases, free order within a phase, narrative | async | multi-lesson | yes | 1 |
| `training` | teacher-quiz game modes | live *or* solo, per mode | 15–30 min | no | 2 |
| `fieldOp` | location or task-list based | async | one session | no | future |

The engine treats all four identically. `activityType` drives presentation only.

**Build order:** build the engine to serve an Operation first. A Case is an Operation with more phases and documents switched on. Agent Training shares the engine but adds a real-time layer — Phase 2.

---

## 3. Top-level shape

```json
{
  "id": "operation-01-the-vault",
  "schemaVersion": "0.4",
  "activityType": "operation",
  "title": "The Vault",
  "subtitle": "Operation 01",
  "status": "draft",

  "theme": { "skin": "field-terminal", "accent": "#FFB000" },

  "curriculum": {
    "framework": "White Rose Maths v3",
    "yearGroups": ["Y6"],
    "blocks": ["Four Operations"],
    "pyp": null,
    "estimatedSessions": 1
  },

  "operation": { "headline": "Four locks. One hour.", "body": "..." },

  "intel": {
    "firstAttemptCorrect": 150,
    "laterAttemptCorrect": 75,
    "hintPenalty": 25,
    "completionBonus": 500,
    "cleanSheetBonus": 300
  },

  "dataset": null,
  "studentSetup": null,
  "documents": [],
  "phases": [],

  "completion": { "message": "Vault open. The ledger is yours, Agent.", "unlocksCertificate": true }
}
```

**Skins.** `skin` is a constrained column with **seven locked archetypes**, listed
in `docs/visual-identity.md` and enforced by a CHECK constraint — an eighth is a
design decision, not a slug you can invent:

`field-terminal` · `case-file` · `archive` · `situation-room` ·
`surveillance-feed` · `field-notebook` · `lab-bench`

`field-terminal` was called `crt-terminal` while this document was drafted; the
column and the CSS use the new name. The teacher dashboard is not a skin at all —
it is the **Bureau face**, and `docs/visual-identity.md` draws that line.

A live session has no `skin` field. A mode declares its own look in code, because
a session is a bank plus a mode rather than an activity; the reasoning is in
`visual-identity.md`.

---

## 4. Intel

Platform-wide and cumulative. Accrues to the agent profile, not the activity.

| field | meaning |
|---|---|
| `firstAttemptCorrect` | correct on the first submission |
| `laterAttemptCorrect` | correct on any subsequent submission |
| `hintPenalty` | deducted per hint revealed, floor of zero |
| `completionBonus` | all non-optional tasks correct |
| `cleanSheetBonus` | activity completed with no wrong answers at all |

Optional tasks carry their own `intelValue` and are the main way strong students earn more.

**Why not speed.** The platform is paper-first: work it out on the sheet, then enter the answer. A speed bonus pays for fast guessing into an unlimited-attempts box and penalises careful long division — the exact behaviour the design exists to build. It also imports noise that has nothing to do with the student: lesson length, fire drills, a teacher pausing to explain. Since Intel is cumulative across a year, that noise would compound permanently. Attempts and hint use give the same spread across a class, measure something real, and can't be gamed by rushing.

If a speed element is ever wanted, it belongs as a per-deployment teacher toggle, default off, awarding a flat bonus under a threshold rather than ranking students — kept out of the platform-wide economy.

### Clearance levels

Platform config, not activity config:

```json
{ "levels": [
  { "id": "junior-agent",   "label": "Junior Agent",   "threshold": 0 },
  { "id": "field-agent",    "label": "Field Agent",    "threshold": 2000 },
  { "id": "senior-analyst", "label": "Senior Analyst", "threshold": 6000 }
]}
```

Set thresholds after watching one class earn real numbers. Do not guess them in advance.

---

## 5. Phases and unlocks

```json
{
  "id": "phase-1",
  "title": "The Ledger",
  "description": "Four suspects. Four sets of figures. One of them is lying.",
  "unlock": { "mode": "always" },
  "taskOrder": "free",
  "tasks": [],
  "reveal": {
    "condition": { "mode": "all", "requires": ["task:p1-t1", "task:p1-t2", "task:p1-t3"] },
    "documentId": "doc-mr-b-dossier",
    "text": "A signature appears at the foot of the page."
  }
}
```

`taskOrder` is `free` (Cases) or `sequential` (Operations).

**Unlock modes:** `always`, `all`, `any`, `count`, `code`. `requires` entries are namespaced `task:<id>`, `phase:<id>`, `document:<id>`.

**Unlocks are agent-level.** Every student walks their own narrative. Unlock state lives on the agent progress record, never on the deployment. One student cracking the cipher does not reveal Mr B to the class.

> Expect students to tell each other anyway. Design reveals so the *document* is the reward, not the secret — a secret will not survive first contact with a Year 6 classroom.

**Open tasks can never appear in a `requires` list.** See section 9.

---

## 6. Tasks

```json
{
  "id": "p1-t2",
  "title": "Invoice 4471",
  "type": "numeric",
  "optional": false,
  "offlineFirst": true,
  "prompt": "Work out the true total on paper, then enter it.",
  "helpText": "Use the declared figure, not the conference note.",
  "inputs": [ { "id": "total", "label": "True total", "prefix": "$", "inputMode": "numeric" } ],
  "answer": { "mode": "static", "values": { "total": 84720 }, "tolerance": 0 },
  "attempts": { "max": null, "feedbackOnWrong": "That doesn't match the ledger. Check your carrying." },
  "hints": [ { "text": "Which figure did Mr D declare, and which did he sign for?" } ],
  "unlocks": ["doc-invoice-4471-annotated"],
  "curriculum": { "block": "Four Operations", "objective": "Long multiplication" }
}
```

### Task types

| type | student does | auto-checked |
|---|---|---|
| `numeric` | enters one or more numbers | yes |
| `text` | enters a short string | yes |
| `choice` | selects from options | yes |
| `ordering` | arranges items into a sequence | yes |
| `selection` | picks n items under constraints | constraints only |
| `assembly` | maps items onto slots, with evidence | **yes** |
| `open` | written work on paper | no |

### Answer modes

`static`, `derived`, `anyOrder`, `crossCheck`.

`crossCheck` powers Operation Cipher Phase 3: the translation route must land on the same coordinates as the reflection route. Two methods, one verification, no extra key.

---

## 7. The `assembly` task — Evidence Board

The replacement for most open tasks. The student maps items onto slots and attaches evidence to each placement. Fully auto-checkable, and it tests reasoning across the whole case rather than recall.

```json
{
  "id": "p3-evidence-board",
  "title": "The Evidence Board",
  "type": "assembly",
  "offlineFirst": false,
  "prompt": "Place each suspect against their role. Attach at least two documents supporting each placement.",

  "slots": [
    { "id": "mastermind", "label": "Mastermind" },
    { "id": "coverup",    "label": "Cover-up" },
    { "id": "accomplice", "label": "Accomplice" },
    { "id": "cleared",    "label": "Cleared" }
  ],

  "items": [
    { "id": "mr-b", "label": "Mr B" },
    { "id": "mr-d", "label": "Mr D" },
    { "id": "mr-k", "label": "Mr K" },
    { "id": "mr-m", "label": "Mr M" }
  ],

  "evidence": {
    "required": 2,
    "pool": "documentsUnlocked",
    "validPerSlot": {
      "mastermind": ["doc-construction-order", "doc-mr-b-dossier", "doc-archive-plan"],
      "coverup":    ["doc-mr-d-profile", "doc-invoice-4471-annotated", "doc-conference-note"],
      "accomplice": ["doc-mr-k-profile", "doc-shipping-manifest"],
      "cleared":    ["doc-mr-m-profile", "doc-phase2-timesheet"]
    }
  },

  "answer": {
    "mode": "static",
    "placements": { "mastermind": "mr-b", "coverup": "mr-d", "accomplice": "mr-k", "cleared": "mr-m" }
  },

  "partialCredit": true,
  "attempts": { "max": null, "feedbackOnWrong": "One or more placements aren't supported by the evidence attached." },
  "intelValue": 400,
  "curriculum": { "block": null, "objective": "PYP — Causation, Perspective" }
}
```

Notes:

- `evidence.pool: "documentsUnlocked"` means a student can only attach documents their own agent record has unlocked. A student who skipped the cipher genuinely cannot cite Mr B's dossier — the mechanic enforces itself.
- `validPerSlot` lists documents that *count* as support. More than `required` are valid; the student picks which to use.
- `partialCredit: true` reports how many placements are correct without saying which. Enough signal to keep working, not enough to brute-force.

**A timeline is an `ordering` task.** Sequencing events across phases needs no new type.

---

## 8. Documents

Cases only. Held separately from tasks so one document can be unlocked by several routes.

```json
{
  "id": "doc-mr-d-profile",
  "title": "Personnel File — Mr D",
  "kind": "profile",
  "availability": "start",
  "body": "Raised a concern in writing on 14 March. Signed the revised figures nine days later.",
  "asset": "/documents/mr-d-profile.jpg",
  "printable": true
}
```

`printable: true` includes the document in the generated PDF dossier, so paper and screen come from one source and cannot drift.

---

## 9. Open tasks

Reserved for genuinely written work. In practice: **one per Case — the Final Brief.** It is the PYP summative assessment, the place a child explains causation and perspective in their own words, and the strongest thing in the design. Everything else that used to be an open task becomes an `assembly` or `ordering` task.

```json
{
  "id": "final-brief",
  "title": "Final Brief",
  "type": "open",
  "offlineFirst": true,
  "prompt": "Using evidence from all three phases, write your Final Brief to the Bureau.",
  "promptPoints": [
    "How did this system work? Who did what?",
    "Who made the decisions — and who was left out?",
    "What were the consequences, intended and unintended?"
  ],

  "selfAssessment": {
    "revealOn": "submitted",
    "exemplar": "A strong brief names who authorised each decision...",
    "checklist": [
      "I named who authorised what, not just who was involved",
      "I used evidence from more than one phase",
      "I explained who was affected beyond Nexus Corp itself"
    ]
  },

  "states": ["notStarted", "submitted", "needsRevisiting", "approved"],
  "intelOn": "approved",
  "intelValue": 600,
  "gating": "never",
  "curriculum": { "block": null, "objective": "PYP — Function, Perspective, Causation" }
}
```

**State machine.** Student moves `notStarted → submitted`. Teacher moves `submitted → approved` or `submitted → needsRevisiting`. From `needsRevisiting` the student resubmits back to `submitted`. Intel is awarded on `approved`, never on submission — otherwise one sentence claims 600 Intel.

**On submission the exemplar and checklist are revealed** so the student gets immediate feedback whether or not the teacher has reached them yet. Self-scoring against a strong model is worthwhile in itself and does not depend on an adult.

**`gating: "never"` is enforced by the engine, not by convention.** An open task must never block progression. Thirty children cannot wait on one adult's availability.

---

## 10. Accounts, schools and access

Four entities:

```
School → Teacher → Deployment → Agent
```

| entity | identity | notes |
|---|---|---|
| School | name, country, domain | enables whole-school licensing later |
| Teacher | Google or Microsoft SSO | adults, work email, standard |
| Deployment | join code | one class run of one activity |
| Agent | codename + PIN | no email, no real name, by default |

```json
{
  "school": { "id": "s_204", "name": "—", "country": "VN", "emailDomain": "—" },

  "teacher": { "id": "t_8812", "schoolId": "s_204", "authProvider": "google" },

  "entitlement": { "teacherId": "t_8812", "activityId": "operation-01-the-vault",
                   "source": "teacherspayteachers", "grantedAt": "2026-09-01T00:00:00Z" },

  "deployment": { "code": "VAULT-7K4", "teacherId": "t_8812",
                  "activityId": "operation-01-the-vault", "className": "6M", "status": "open" },

  "agent": { "id": "a_44120", "codename": "Marlow", "pinHash": "...",
             "teacherId": "t_8812", "intelTotal": 3400, "clearance": "field-agent" }
}
```

**Teacher SSO: yes.** Google and Microsoft. Straightforward.

**Student SSO: opt-in per school, never the default path.** School Google Workspace and Microsoft 365 tenants block third-party sign-in until an administrator approves the app. A teacher who buys on Sunday and teaches Monday would need an IT ticket first — which is how you lose a teacher who has already paid. Default is join code plus codename plus PIN, exactly as Kahoot and Blooket do it. Student SSO becomes a feature a school enables, and a selling point for whole-school licences.

Codename plus PIN also holds nothing identifying, which keeps COPPA, the UK Age Appropriate Design Code and school data processing agreements out of scope until you choose to take them on.

Entitlements are permanent and account-level: one purchase, unlimited deployments. An agent joins a *deployment*, so the same student can run the same Operation next year without old progress interfering. Intel persists on the agent across deployments.

---

## 11. Progress record

```json
{
  "agentId": "a_44120",
  "deploymentCode": "VAULT-7K4",
  "selections": { "countries": ["Peru", "Norway", "Kenya", "Vietnam"] },
  "tasks": {
    "lock-1": { "status": "correct", "attempts": 1, "hintsUsed": 0, "intelEarned": 150 },
    "lock-2": { "status": "correct", "attempts": 3, "hintsUsed": 1, "intelEarned": 50 },
    "final-brief": { "status": "needsRevisiting", "submittedAt": "...", "selfScore": 2 }
  },
  "documentsUnlocked": ["doc-mr-b-dossier"],
  "completedAt": null
}
```

`attempts` and `hintsUsed` per task, aggregated across a class, are the teacher dashboard's entire diagnostic value — and they come free.

---

## 12. Datasets and personalised answers

Only Top Trumps needs this. Deliberately narrow.

```json
"dataset": {
  "id": "countries-61",
  "keyField": "name",
  "publicFields": ["name", "population", "landArea", "gdpPerCapita", "lifeExpectancy", "internetUsers"],
  "keyOnlyFields": ["population3sf", "landArea3sf", "gdpPerCapita2sf", "gdpTotal"]
},
"studentSetup": {
  "selections": [
    { "id": "countries", "count": 4, "from": "countries-61",
      "constraints": [ { "rule": "oneFromEachBand", "field": "band" } ] }
  ]
}
```

`publicFields` reach the browser. `keyOnlyFields` never do.

```json
"answer": { "mode": "derived", "values": { "pop": "selection.countries[0].population3sf" } }
```

```json
"answer": { "mode": "derived",
            "values": { "meanLife": "mean(selection.countries[*].lifeExpectancy)" },
            "tolerance": 0.03 }
```

Permitted functions: `mean`, `sum`, `product`, `ratio`, `roundSf`, `roundDp`, `abs`. Nothing else.

**This is the schema's one sharp edge**, and the reason Top Trumps migrates last. Everything else works without it existing.

---

## 13. Resolved decisions

| decision | answer |
|---|---|
| Codename collisions | Block at creation, scoped per teacher. Offer a generator as the default path. |
| Case phase unlocking | Agent-driven by progress. Teacher sets an optional phase ceiling on the deployment (default: off) to prevent students reaching untaught content. |
| Progress reset | Not needed. PIN reset (teacher-triggered), delete stray agent, and test-agent reset in dev are the three real needs. |
| School assignment | Teacher creates school at signup via search-first flow — type name, see matches, create only if none fit. |
| Speed bonus | Off the platform. Attempts and hint use are the differentiators. Speed may be a per-deployment teacher toggle in future, default off. |
| Student SSO | Opt-in per school, never the default path. Codename + PIN is always the baseline. |
| Open tasks gating | `gating: "never"` enforced by the engine. Open tasks cannot block progression. |
| Intel on open tasks | Awarded on teacher `approved`, not on student submission. |

## 14. Remaining open questions

1. **Codename generator word list.** Who curates it, and how is it filtered? Must be done before first pilot.
2. **AI-assisted question bank authoring.** From a topic or pasted text,
   teacher-reviewed before publish. What model, what review UI? *Still open* —
   the manual editor and file import both shipped 2026-08-31; this did not.
3. **Agent Training Intel.** Per-correct-answer plus a net-token bonus at game
   end. Thresholds TBD after a real session. *Partly settled 2026-08-31:* tokens
   convert to Intel, and the podium bonus is gone with the leaderboard.

---

## 15. Agent Training schema

Two objects: a `QuestionBank` (content, mode-agnostic) and a `TrainingSession`
(mode + bank + deployment config). A bank built for one mode runs in any other
without rebuilding the questions.

**What shipped, and what this section describes.** Agent Training has two modes
and they arrived in the opposite order to the one drafted here.

**Signal Check** is built and live, and is the only mode the marketing site
shows. Live, whole-class, synchronous: one question to everyone at once, a fixed
answer window, three question types and **no speed bonus** — a **top-three
podium** was added 2026-09-01, with full results to the teacher. It went first deliberately — it is the thinner of the two, and its job
was to prove the realtime transport that both modes need.

**Mainframe Breach is next**, and the JSON below is its intended design, kept
rather than rewritten. Two things about it have moved since it was drafted and
the config is corrected to match:

- **It is single player** (decided 2026-08-31). The multiplayer problem is solved
  once, in Signal Check, rather than inside a mode that also has a game to
  design. It keeps its token economy and its fiction; attacks and defences run
  against simulated nodes, as Solo Practice already does.
- **`podiumBonus` is gone; the podium came back without it.** *(Reversed
  2026-09-01 — see `docs/question-banks.md`.)* Signal Check now ends on a top
  three, with every result to the teacher and three names to the room. The
  *bonus* stays gone: it awarded extra Intel for placing, which is a scoring
  change, where the podium is a display of scores already kept.

`docs/question-banks.md` is the fuller account of both modes.

```json
{
  "id": "qb-four-operations-Y6-01",
  "title": "Four Operations — Year 6",
  "teacherId": "t_8812",
  "generatedFrom": "ai-assisted",
  "reviewedAt": "2026-09-01T00:00:00Z",
  "curriculum": {
    "framework": "White Rose Maths v3",
    "blocks": ["Four Operations"],
    "yearGroups": ["Y6"]
  },
  "questions": [
    {
      "id": "q-001",
      "prompt": "What is 48 × 37?",
      "options": ["1,776", "1,736", "1,756", "1,816"],
      "answer": "1,776",
      "difficulty": 2,
      "curriculum": { "objective": "Long multiplication" }
    }
  ]
}
```

```json
{
  "id": "training-01-mainframe-breach",
  "schemaVersion": "0.4",
  "activityType": "training",
  "mode": "mainframe-breach",
  "title": "Mainframe Breach",
  "subtitle": "Agent Training — Simulation 01",
  "status": "draft",
  "theme": { "skin": "field-terminal", "accent": "#55FF55" },
  "curriculum": {
    "framework": "White Rose Maths v3",
    "yearGroups": ["Y6"],
    "blocks": ["Four Operations"],
    "estimatedSessions": 1
  },
  "questionBankId": "qb-four-operations-Y6-01",
  "sessionConfig": {
    "questionCount": 15,
    "timeLimitPerQuestion": null,
    "attacksEnabled": true,
    "firewallsEnabled": true,
    "deepScanEnabled": true
  },
  "intel": {
    "perCorrectAnswer": { "first": 50, "later": 25 },
    "netTokenBonus": { "per100tokens": 10 }
  }
}
```

`timeLimitPerQuestion: null` means no timer — the teacher can set a value when
ready. Matches design rule 4.

Access tokens **convert to Intel** at the end of a session (decided 2026-08-31),
which `netTokenBonus` is the mechanism for. Whether *earned* or *unspent* tokens
convert is still open: converting only the remainder pays a child for not
spending, which is the mechanic the mode is built on. See
`docs/intel-and-clearance.md`.

---

## 16. Fit test

Map these onto the schema on paper before writing engine code.

- [ ] Operation 01 — five sequential locks, Intel by attempts, clean-sheet bonus
- [ ] Operation Cipher Phase 1 — four suspect files, `taskOrder: free`, optional cipher unlock, agent-level document unlocks
- [ ] Operation Cipher Phase 3 — coordinates, double reflection, `crossCheck`, code-gated reveal
- [ ] Operation Cipher endgame — `assembly` Evidence Board, `ordering` timeline, one `open` Final Brief with self-assessment
- [ ] Top Trumps Stage 1 — `selection` with band constraints
- [ ] Top Trumps Stage 6 — `derived` answer using `mean`

Anything needing a field this document doesn't define: change the schema, then the code.
