---
story: "02"
title: Teacher profile wizard — implementation tasks
status: draft
---

# Story 02 — Implementation tasks

> Tasks for [Story 02](../02-teacher-profile-wizard.md). Depends on [Story 01](../01-entry-point.md).

## Data model

- [ ] `instructor_profile` model: name/pseudo, email, bio, github username, residency, audience,
      experience, taught-environments (multi), socials (LinkedIn/Nostr/X), free-text.

## Wizard UI

- [ ] Step-by-step form with the required + optional fields.
- [ ] Prefill email from the existing database; show it (read-only or editable — decide).
- [ ] Conditional "environments taught" multiselect, shown only if years-of-experience is set.

## Validation

- [ ] Bio length 50–300 words; editable later.
- [ ] GitHub username format check.
- [ ] Confirmation-email field is **required** (DECIDED, PRD OQ-7 / FR-10) — validate it matches the email.

## Persistence

- [ ] Save profile and allow continue to the next onboarding step (course submission).

## Done when

- [ ] A user completes the profile and proceeds to [Story 03](../03-course-submission.md).
