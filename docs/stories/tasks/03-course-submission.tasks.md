---
story: "03"
title: Course submission — implementation tasks
status: draft
---

# Story 03 — Implementation tasks

> Tasks for [Story 03](../03-course-submission.md). Depends on [Story 02](../02-teacher-profile-wizard.md).
> **Artifact pipeline — DECIDED (architecture, resolves OQ-3):** the wizard generates
> `MKT_BRIEFING.md` from submission data and **scaffolds** `COURSE_MASTER_PLAN.md` (the
> instructor then authors/edits it as Markdown in the `courses` repo); a `course.yml` companion
> carries machine metadata. See architecture.md (Artifact Pipeline).

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

- [ ] Generate `MKT_BRIEFING.md` from the wizard submission data.
- [ ] Scaffold `COURSE_MASTER_PLAN.md` from the collected data (shape in the story) for the
      instructor to author/edit in GitHub.
- [ ] Emit the `course.yml` companion (machine metadata: category, cohort, shortname, type).
- [ ] Hand the package off to the GitHub PR flow ([Story 05](../05-approval-github-pr.md)).
- [ ] **On submit, intranet course content becomes read-only (FR-17a):** further edits happen
      as Markdown in the `courses` repo; the intranet keeps the record but stops being the
      content-editing surface.

## Done when

- [ ] A submission yields a structured `.md` package ready to open as a PR.
