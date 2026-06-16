---
story: "03"
title: Course submission — implementation tasks
status: draft
---

# Story 03 — Implementation tasks

> Tasks for [Story 03](../03-course-submission.md). Depends on [Story 02](../02-teacher-profile-wizard.md).
> **Deferred to architecture (OQ-3):** pin the single generation point for the `.md` package /
> COURSE_MASTER_PLAN / MKT_BRIEFING (wizard / approval / loader) before building §4.6 —
> [risks.md](../../risks.md#artifact-generation-pipeline).

## Data model

- [ ] `course_submission` model: course name, type (MOOC | cohort), language, elevator pitch,
      problem, 3 objectives, ideal student, ownership/licensing.
- [ ] Cohort-only fields: start/end date, classes count, recurrence, time, time zone.
- [ ] Per-class breakdown: class number/title, topics, objectives, activities (quiz/exercise),
      source materials.

## Wizard UI (Part A — metadata)

- [ ] Conditional branch: MOOC → show language; Cohort → show schedule fields.
- [ ] Validation/help text (elevator pitch, ~2h/class recurrence hint, licensing checkboxes).
- [ ] Video(s) upload for social presentation.
- [ ] **Asset handling (FR-16a):** define accepted types, size/count limits, and storage for
      videos / source materials / images, and how their references survive submission → GitHub
      Markdown → Moodle (limits TBD with the team).

## Wizard UI (Part B — content breakdown)

- [ ] Repeatable per-class editor producing the agreed structure.

## Package generation

- [ ] Render the review-ready Markdown template from the collected data (shape in the story).
- [ ] Create/seed the `MKT_BRIEFING` `.md` in GitHub and return the link to the instructor.
- [ ] Hand the package to the GitHub PR flow ([Story 05](../05-approval-github-pr.md)).
- [ ] **On submit, intranet course content becomes read-only (FR-17a):** further edits happen
      as Markdown in the `courses` repo; the intranet keeps the record but stops being the
      content-editing surface.

## Done when

- [ ] A submission yields a structured `.md` package ready to open as a PR.
