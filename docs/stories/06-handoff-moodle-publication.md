---
id: "06"
title: Handoff to Moodle, publication & instructor profile
track: product
status: draft
mvp: true
owner: TBD
depends_on: ["05", "PA", "PB", "PC", "P101"]
risk: high
jira: ENG-277
---

# Story 06 — Handoff to Moodle, publication, and instructor profile

**Tasks:** [implementation breakdown →](tasks/06-handoff-moodle-publication.tasks.md)

> **Critical path.** This is the only point where real value is delivered (a live course in
> Moodle), and it depends on the *entire* platform track (PA/PB/PC/P101), which is currently
> undefined. Spike the integration before committing to dates. See [roadmap.md](../roadmap.md).

## User story

**As** an approved instructor, **I want** my course published and my profile marked as an
instructor, **so that** students can find me.

## Acceptance criteria

- **Document generation** — On approval, generation of the `MKT_BRIEFING` and the
  `COURSE_MASTER_PLAN` ([PA](../platform/PA-document-generation.md)) is triggered following
  the temporal sequence ([PB](../platform/PB-handoff-sequence.md)).
- **Handoff** — The PR merge **automatically** triggers the handoff (no manual Sheet step); when
  the `COURSE_MASTER_PLAN` is ready the loader builds the course
  ([PC](../platform/PC-loader-build-course.md)), and the intranet records the handoff event and
  notifies Operations. The build may run async/queued.
- **Handoff confirmation** — The intranet records the Moodle Link (write-back) and reflects
  the course as published.
- **Instructor role (intranet)** — With ≥1 approved course, the profile shows the instructor
  indicator.
- **Teacher role (Moodle)** — Assigned by the loader ([P101](../platform/P101-moodle-user-role-enrolment.md)).
- **Course visible in its category** — After publishing, it sits in the category assigned by
  the reviewer.

## Notes

- Sub-state: `approved` (Ops ok) → `published` (loader confirmed, Moodle Link exists). The
  handoff is **not instantaneous** (loader is batch).
- The "instructor marker" lives in the intranet (own data: ≥1 approved course).
- The structure Google Doc is **no longer** a manual bridge: the `COURSE_MASTER_PLAN` is the
  loader's input, which builds the structure automatically ([PC](../platform/PC-loader-build-course.md)).

## Resolved

- **Dual handoff mechanism** — RESOLVED (PRD OQ-1): PR merge triggers the build automatically;
  no manual Sheet. Reflected in Acceptance criteria above and in [Story 05](05-approval-github-pr.md).
- **Post-publish edits / data consistency** — RESOLVED (PRD OQ-2): fire-and-forget. After publish,
  content lives only in Moodle (edits there); the intranet keeps the historical record only.
