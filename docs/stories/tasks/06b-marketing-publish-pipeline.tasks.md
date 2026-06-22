---
story: "06b"
title: Publish & course promotions (marketing) — implementation tasks
status: draft
---

# Story 06b — Implementation tasks

> **Out of v1 scope (deferred, `mvp: false`).** Post-MVP per PRD §8 (FR-42). The tasks below
> are not part of the 3-month v1 skeleton; kept for dependency visibility.
>
> Tasks for [Story 06b](../06b-marketing-publish-pipeline.md). Owner: Aria Cediel.
> Depends on `MKT_BRIEFING` ([PA](../../platform/PA-document-generation.md)) and the
> `published` state ([Story 06](../06-handoff-moodle-publication.md)).

## Trigger 1 — `approved`

- [ ] Notify Aria with the `MKT_BRIEFING` link when a course becomes `approved`.

## Draft creation (Metricool)

- [ ] Integrate Metricool: create post drafts for X and LinkedIn (copy, image, hashtags),
      **without** a scheduled date yet.
- [ ] Specify the Canva → Metricool asset step end to end (overlaps [Story 08](../08-public-instructor-profile.md)).

## Trigger 2 — `published`

- [ ] On Moodle Link existing, auto-update the Nostr draft with the live link and schedule
      same-day posting.

## Nostr exception

- [ ] Flag Nostr in the briefing as a **manual** post (Metricool has no native Nostr).

## Status & cadence

- [ ] Reflect "promo ready" (after drafts) and "promo published" (after scheduling) in the Sheet/intranet.
- [ ] **Decide** posting cadence when several courses land the same day (proposal: max 1/day, FIFO).

## Done when

- [ ] An approved course produces reviewable drafts; a published one is scheduled same-day.
