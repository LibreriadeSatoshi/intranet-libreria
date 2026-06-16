---
id: "02"
title: Teacher profile wizard
track: product
status: draft
mvp: true
owner: TBD
depends_on: ["01"]
risk: medium
jira: ENG-274
---

# Story 02 — Teacher profile wizard

**Tasks:** [implementation breakdown →](tasks/02-teacher-profile-wizard.tasks.md)

## User story

**As** an aspiring teacher, **I want** a step-by-step form to submit my profile, **so that** my
profile will be easy and frictionless.

## Scope

**Covers:**
- Creating a profile for users who aspire to become instructors/teachers on the platform.
- Allowing applicants to continue to the next step of the onboarding process.

**Does not cover:**
- Course content information.
- Collection of course metadata (→ [Story 03](03-course-submission.md)).
- Generating a review-ready course package compatible with the GitHub review workflow
  (→ [Story 03](03-course-submission.md) / [Story 05](05-approval-github-pr.md)).

## Required fields

- **Name or Pseudo** — public to everyone.
- **Email** — should show the one in our database.
- **Confirmation Email** — should show the one in our database — **required**; must match the email.
- **Biography / short bio** — 50 to 300 words. Editable later.
- **GitHub username.**

**Social media section** (crucial for students to get to know you):

- LinkedIn — *Optional*
- Nostr — *Optional*
- X — *Optional*
- Years of teaching experience — *Optional*
- Environments where you have taught (multiple selection, only if the field above is filled):
  - [ ] Online
  - [ ] In-Person
  - [ ] University Campuses
  - [ ] None of the above
- Do you have an audience with whom you share your knowledge?
- Where is your residency? (needed for payments)
- Anything else you want to share with us — free text block.

## Resolved

- **Confirmation Email** — required (PRD OQ-7 / FR-10). Reflected in Required fields above.
