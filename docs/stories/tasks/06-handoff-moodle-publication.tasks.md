---
story: "06"
title: Handoff to Moodle & publication — implementation tasks
status: draft
---

# Story 06 — Implementation tasks

> Tasks for [Story 06](../06-handoff-moodle-publication.md). **Critical path.**
> Depends on the entire platform track (PA/PB/PC/P101), all **undefined today** —
> [risks.md](../../risks.md#phantom-platform-stories). **Spike before dating.**

## Platform pre-reqs (define first)

- [ ] [PA](../../platform/PA-document-generation.md) — generate `MKT_BRIEFING` + `COURSE_MASTER_PLAN` on approval.
- [ ] [PB](../../platform/PB-handoff-sequence.md) — define the temporal handoff sequence.
- [ ] [PC](../../platform/PC-loader-build-course.md) — loader builds the course from `COURSE_MASTER_PLAN`.
- [ ] [P101](../../platform/P101-moodle-user-role-enrolment.md) — Moodle user/role/enrolment functions.

## Handoff

- [ ] On approval/merge, **automatically** trigger document generation (PA) following the
      sequence (PB) — **no manual Sheet step** (DECIDED, OQ-1).
- [ ] Intranet records the handoff event and **notifies Operations** that the handoff started
      (FR-37a).
- [ ] Loader builds the course (PC) — triggered by the merge from [Story 05](../05-approval-github-pr.md);
      the build itself may run async/queued.

## Write-back & state

- [ ] Record the Moodle Link in the intranet (write-back); flip course to `published`.
- [ ] Model the `approved` → `published` sub-state (handoff is batch, not instantaneous).

## Roles & visibility

- [ ] Set the intranet **instructor marker** when the user has ≥1 approved course.
- [ ] Loader assigns the Moodle **Teacher** role (P101).
- [ ] Course lands in the category assigned by the reviewer.

## Done when

- [ ] An approved course becomes a live (hidden) Moodle course with its link recorded back.
