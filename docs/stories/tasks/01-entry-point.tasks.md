---
story: "01"
title: Entry point — implementation tasks
status: draft
---

# Story 01 — Implementation tasks

> Tasks for [Story 01](../01-entry-point.md). Depends on auth ([Story 00](../00-federated-identity.md)).

## UI

- [ ] Add a "Become a Teacher" CTA in a consistent, documented location (define exact placement).
- [ ] Wire the CTA into the public landing on Librería's site.

## State-aware routing

- [ ] Not logged in → delegate to login (Story 00).
- [ ] Logged in, no submission → start a new draft.
- [ ] Has draft(s) → submissions dashboard.
- [ ] Already an instructor → instructor dashboard.

## Visibility / authorization

- [ ] Decide and implement who sees the entry point (open question).
- [ ] Enforce the intranet role model — instructor / reviewer-Ops / marketing / **admin** —
      per the role × capability matrix (FR-7a, least-privilege) for what each role sees/does.

## Done when

- [ ] From a single CTA, each user state lands on the correct destination.
