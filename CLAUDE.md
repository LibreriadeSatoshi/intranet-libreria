# CLAUDE.md — lds-intranet

Instructions for any AI/dev working this repo. This file **routes**; it does not
restate the specs. When in doubt, the sources of truth below win over this file.

## What this is

Standalone intranet that owns the course-submission lifecycle (instructor onboarding →
wizard → drafts → GitHub-PR approval → handoff that builds the course in Moodle). The
intranet is the source of truth for intake/approval; Moodle is the post-handoff destination.
Status: planning → building v1.

## Source of truth (read in this order, respect verbatim)

1. `_bmad-output/planning-artifacts/prds/prd-lds-intranet-2026-06-16/prd.md` — the WHAT
   (FRs, NFRs, glossary §3, status lifecycle §4.5, role matrix §4.1, scope §8).
2. `_bmad-output/planning-artifacts/architecture.md` — the HOW (stack, project structure,
   implementation patterns, naming, handoff orchestration). Also its sibling `addendum.md`.
3. `docs/stories/*.md` + `docs/stories/tasks/*.tasks.md` — **scope checklist only**, not the
   build contract. They are thin pointers back to (1) and (2).

Ranking: PRD + architecture are authoritative. Stories/tasks tell you WHICH work and WHEN,
not HOW. If a task seems to conflict with architecture, architecture wins — flag the task.

## Ignore (do not read as truth)

- `lds-intranet.md` — legacy master doc (old numbering, partial duplicates). Superseded by
  the PRD. Do not build from it.
- Any `docs/stories/tasks/*` file with an "Out of v1 scope (deferred)" banner.

## Scope rules (v1)

- Build only stories with `mvp: true` in front-matter: **00, 01, 02, 03, 04, 05, 06**.
- Do NOT build: 06b, 07, 08, 09 (deferred), and any task tagged `[later]`
  (e.g. FR-21 multi-draft, FR-7 instructor migration).
- customer.io notifications (§4.7) are v1 but have no dedicated story yet — their touchpoints
  are threaded into tasks 01/05/06. Do not invent the integration; ask before building it.

## Open questions — STOP, do not resolve

If a task says "blocked by OQ-X" or a value is "TBD", do not guess. Pause and ask the user.
- **OQ-4** — GitHub ↔ canonical-identity mapping + existing-student→teacher linking. Build-blocking.
- **OQ-6** — Moodle loader contract (PA/PB/PC/P101). Resolved by the spike (see Build order).
- **FR-16a** — asset type/size/count limits. TBD with the team.

## Build order

1. **Spike the `moodle-course-loader` integration first** (resolves OQ-6) — highest priority.
2. Scaffold the app, then implement by dependency: 00 → 01 → 02 → 03 → 04 → 05 → 06.
3. Wire customer.io touchpoints as the relevant transitions are built.
Per story: implement, then validate against that story's Acceptance criteria before moving on.

## Tech stack (pointer — do not restate or change)

Defined in architecture.md (Starter Template + Core Architectural Decisions): create-t3-app
(Next.js App Router + TypeScript + tRPC + Prisma + Auth.js + Tailwind), PostgreSQL, Authentik
(IdP), pg-boss (queue), Garage (S3), the Python `moodle-course-loader` (separate repo),
customer.io. Self-hosted via Docker Compose. Once scaffolded, `package.json` is the live
record — do not swap libraries without updating architecture.md first.

## Hard conventions (enforced — detail in architecture.md §Implementation Patterns)

- **Glossary + enum strings verbatim** (PRD §3 and the §4.5 status strings) in code, DB values,
  event names, and UI. No synonyms, no re-casing.
- **One status authority:** every status change goes through `transitionSubmission()` and emits
  a `LifecycleEvent` (append-only). Never write `Submission.status` directly anywhere else.
- **Validation once:** define each Zod schema in `src/lib/schemas/` and reuse it client + server.
- **Env only via `src/env.js`** (`@t3-oss/env`). Never `process.env` directly.
- **Moodle only from jobs:** never call Moodle on the request path; handoff runs in pg-boss jobs.
- **Verify all webhook signatures** before doing work; respond fast, enqueue the rest.
- Feature-first layout under `src/features/`; integrations under `src/server/`.

## Working agreements

- Language: English for spec, code, UI (NFR-6). Instructor-authored content may be ES/EN.
- **No emoji or icons inside files** (docs, code, commits). Plain text only.
- Git: work on a branch and open a PR; do not commit straight to `main` (multiple devs).
  PR titles describe the change; the body must explain any large deletion.
- Don't duplicate spec content into new files. Add a pointer, not a copy — duplication is how
  this project drifts.
