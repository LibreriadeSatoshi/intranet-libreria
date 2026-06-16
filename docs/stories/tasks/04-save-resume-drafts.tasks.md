---
story: "04"
title: Save & resume drafts — implementation tasks
status: draft
---

# Story 04 — Implementation tasks

> Tasks for [Story 04](../04-save-resume-drafts.md). Cross-cuts the wizard
> ([02](../02-teacher-profile-wizard.md)–[03](../03-course-submission.md)).
> **MVP note:** basic persistence is skeleton; multi-draft + fine-grained autosave can defer
> ([roadmap.md](../../roadmap.md)).

## Persistence

- [ ] Autosave per step on "Next" / "Save draft", server-side against the account.
- [ ] Store `status = draft`; never delete states.
- [ ] **Capture lifecycle events with timestamps from day 1 (FR-22, now v1):** wizard started,
      step reached, submitted, changes-requested, approved, published, rejected — raw events
      only (analytics dashboards remain `[later]`).
- [ ] Each draft is an independent row tied to the user.

## Resume logic

- [ ] Compute and return to the "last incomplete step" with prior steps pre-filled.
- [ ] Gate: a draft does not enter review until explicit submission.

## Dashboard

- [ ] List all of a user's drafts, each with a "Continue" action.
- [ ] Support multiple simultaneous drafts of different courses.

## Done when

- [ ] A user resumes any draft across sessions at the right step; submission is explicit.
