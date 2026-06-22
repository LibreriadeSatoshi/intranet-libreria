---
story: "08"
title: Public instructor profile, featured & exports — implementation tasks
status: draft
---

# Story 08 — Implementation tasks

> **Out of v1 scope (deferred, `mvp: false`).** Post-MVP per PRD §8 (FR-43; public front). The
> tasks below are not part of the 3-month v1 skeleton; kept for dependency visibility.
>
> Tasks for [Story 08](../08-public-instructor-profile.md). Owner: Aria Cediel / Marketing.
> Depends on `published` courses ([Story 06](../06-handoff-moodle-publication.md)) and
> `MKT_BRIEFING` ([PA](../../platform/PA-document-generation.md)).

## Public surfaces ("entity, two views")

- [ ] Public instructor profile page listing the instructor's approved courses.
- [ ] Public course landing rendered from wizard data, linking to the Moodle Link.
- [ ] Share actions per course: copy link + ≥1 social network.

## Featured

- [ ] Add a `featured` flag on the course model (admin-set).
- [ ] Render a featured section surfacing flagged courses.

## Marketing role & inputs

- [ ] Add the **marketing** intranet role (4th role) with access to courses for promotion.
- [ ] Give marketing access to the generated `MKT_BRIEFING` as an intake input.

## Exports

- [ ] Structured export of each course's wizard fields (title, description, audience,
      requirements, curriculum, bio, code) for external tools.
- [ ] Per-course self-contained `.md` export (serializes existing data; captures nothing new).

## Done when

- [ ] Courses have public profile/landing + share, admins can feature, marketing can export.
