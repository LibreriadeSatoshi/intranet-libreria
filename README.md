# Intranet Librería de Satoshi

Librería de Satoshi's intranet — source of truth for instructor onboarding and
course submission, with handoff to Moodle.

> **Status:** Planning / pre-implementation. No application code yet — this repo holds the
> product specification (PRD), the architecture decision, and the per-story breakdown.

## Context

The intranet is a standalone app (independent of Moodle) that owns the course
submission lifecycle: from a user deciding to teach, through a wizard and drafts,
to a GitHub-based approval workflow run by Operations, and finally the handoff to
Moodle. After handoff, content lives in Moodle and the intranet keeps the record
as history (basis for future analytics).

- The intranet is the **source of truth** for intake and approval.
- Moodle = post-handoff execution / final destination of the content.
- Identity is federated through a dedicated IdP (OIDC); see Story 00.
- The public front (instructor profiles, course landing pages) is **deferred to phase 2**
  (story 08); v1 ships the private workflow only.

## Sources of truth (read these first)

1. **PRD** — `_bmad-output/planning-artifacts/prds/prd-lds-intranet-2026-06-16/prd.md`
   (the what: FRs, scope, glossary, status lifecycle).
2. **Architecture** — `_bmad-output/planning-artifacts/architecture.md`
   (the how: stack, structure, patterns).
3. **`CLAUDE.md`** (repo root) — working instructions for any AI/dev; routes to the above.
4. **`docs/stories/` + `docs/stories/tasks/`** — scope checklist (thin pointers to 1–2).

## Stories (epic ENG-271)

Story files live in `docs/stories/` (numbered 00–09; `06b` and `09` are extras).
Jira mapping (current — the older ENG-272..278 sequence is superseded; one ticket per
story file under the epic):

| File | Story | Jira | v1 |
|---|---|---|---|
| `00-federated-identity` | Federated identity (IdP) | ENG-295 | yes |
| `01-entry-point` | "Teach on LdS" entry point | ENG-296 | yes |
| `02-teacher-profile-wizard` | Teacher profile wizard | ENG-297 | yes |
| `03-course-submission` | Course submission | ENG-298 | yes |
| `04-save-resume-drafts` | Save & resume drafts | ENG-299 | yes |
| `05-approval-github-pr` | Approval via GitHub PR | ENG-300 | yes |
| `06-handoff-moodle-publication` | Handoff to Moodle & publication | ENG-301 | yes |
| `06b-marketing-publish-pipeline` | Marketing publish pipeline | — | deferred |
| `07-instructor-promotion-tools` | Instructor promotion kit | — | deferred |
| `08-public-instructor-profile` | Public profile, featured & exports | — | deferred |
| `09-doc-reference` | Teacher help / reference docs | — | deferred (stub) |

**Cross-cutting work (no story file — born from the architecture decision):**

| Work | Purpose | Jira | v1 |
|---|---|---|---|
| Project init | `create-t3-app` scaffold + Docker Compose (Postgres/Authentik/Garage/loader); prerequisite, blocks all stories | TBD | yes |
| Moodle loader spike | Validate `moodle-course-loader`, define the platform track (PA/PB/PC/P101); resolves OQ-6/G1, blocks Story 06 | TBD | yes |
| Identity linking design | Email-match confirmation UX + student→teacher fallback + PR-author↔`sub` (gap G2); design step inside Story 00 | within ENG-295 | yes |

Epic: [ENG-271 — Instructor onboarding & course submission](https://b4os.atlassian.net/browse/ENG-271).

## Related repositories

- `moodle-course-loader` — Python CLI that builds courses in Moodle via the Web
  Services API (consumes the `COURSE_MASTER_PLAN`).
- `courses` — the GitHub repo where course submissions live as Pull Requests
  (Story 05 review workflow).

## Repository layout

```
CLAUDE.md       Working instructions for AI/devs (routes to the sources of truth)
docs/           Product specification: stories, tasks, risks, roadmap, platform track
_bmad-output/   PRD + architecture decision (planning artifacts)
```
