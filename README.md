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

Story files live in `docs/stories/` (numbered 00–09; `06b` and `09` are extras). The
master Jira sequence (ENG-27x) is an older, coarser mapping — several files share one
Jira id. Mapping:

| File | Story | Jira | v1 |
|---|---|---|---|
| `00-federated-identity` | Federated identity (IdP) | ENG-272 | yes |
| `01-entry-point` | "Teach on LdS" entry point | ENG-273 | yes |
| `02-teacher-profile-wizard` | Teacher profile wizard | ENG-274 | yes |
| `03-course-submission` | Course submission | ENG-274 | yes |
| `04-save-resume-drafts` | Save & resume drafts | ENG-275 | yes |
| `05-approval-github-pr` | Approval via GitHub PR | ENG-276 | yes |
| `06-handoff-moodle-publication` | Handoff to Moodle & publication | ENG-277 | yes |
| `06b-marketing-publish-pipeline` | Marketing publish pipeline | — | deferred |
| `07-instructor-promotion-tools` | Instructor promotion kit | ENG-278 | deferred |
| `08-public-instructor-profile` | Public profile, featured & exports | — | deferred |
| `09-doc-reference` | Teacher help / reference docs | — | deferred (stub) |

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
