---
id: "04"
title: Save & resume drafts
track: product
status: draft
mvp: true
owner: TBD
depends_on: ["02", "03"]
risk: medium
jira: ENG-275
---

# Story 04 — Save & resume drafts

**Tasks:** [implementation breakdown →](tasks/04-save-resume-drafts.tasks.md)

> Cross-cutting: this story spans the wizard ([02](02-teacher-profile-wizard.md)–[03](03-course-submission.md)).
> Kept as its own unit with explicit dependencies rather than scattered across the wizard
> stories.

## User story

**As** an instructor, **I want** my submission saved as a draft, **so that** I can finish it
across multiple sessions.

## Acceptance criteria

- **Autosave per step** — "Next" or "Save draft" persist server-side against my account.
- **Resume** — I return to the last incomplete step with prior steps pre-filled.
- **Draft visible** — In my dashboard with status `draft` and a "Continue" action.
- **No partial submission** — A draft does not enter review until explicit submission.
- **Multiple drafts** — Several drafts of different courses simultaneously.
- **Drafts listed** — The dashboard shows all of them, each with "Continue".

## Notes

- The draft is an intranet-owned entity (`submission` / `course_draft`). It is **no longer a
  hidden Moodle course**.
- This is real development (persistence, model, "last incomplete step" logic), not
  configuration.
- States are not deleted, they are recorded: basis for analytics (wizard abandonment). Don't
  build the analytics now — just don't destroy the trail.
- Each draft is an independent row tied to the user.

## MVP note

Basic persistence is part of the skeleton, but **multi-draft + fine-grained autosave** can be
deferred. See [roadmap.md](../roadmap.md).
