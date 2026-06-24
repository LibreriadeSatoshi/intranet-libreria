# Addendum — libreria-intranet PRD

Technical depth, mechanism options, and source-doc detail that informs downstream work (architecture, solution design) but does not belong in the PRD body. Not authoritative for *what* to build (the PRD is) — this captures *how*-leaning material and rationale so it isn't lost.

## A. Technology choices named in the source docs

- **Identity:** Self-hosted **Authentik** as the IdP; Google + GitHub via standard OAuth/OIDC; Moodle 5.1 as an OIDC client. Local username/password as the third method. One-time migration of existing instructors to the canonical `sub` (deferred). `wstoken` is a separate Moodle web-service auth plane, distinct from the OIDC login token.
- **Approval transport:** GitHub PRs in a single `courses` repo. GPG-signed commits desirable (aligns with the Bitcoin ecosystem).
- **Course build:** `moodle-course-loader` (Python). Existing, proven machinery: `PlanBCourseBuilder` + `planb_source` — parses semi-structured Markdown (h1 → Parts, h2 → Chapters), creates Moodle sections/pages, uploads assets, renders HTML, embeds video. Current web-service coverage: `core_course_*`, file upload, set course image. **Missing:** user/role/enrolment functions (added by platform P101).
- **YAML note:** The content contract is the Markdown **COURSE_MASTER_PLAN**, *not* YAML emitted by the intranet. The loader generates internal YAML from the Plan using the Claude API.

## B. Handoff mechanism — DECIDED (OQ-1, 2026-06-16)

The source docs described two architectures as if one:
1. **Story 05:** PR merge → **GitHub Action** → Moodle (instantaneous automation).
2. **Story 06 / PB:** COURSE_MASTER_PLAN reference → **Sheet** → **batch loader** run → Moodle (asynchronous).

**Decision (the chosen hybrid):** **PR merge triggers the build automatically** (mechanism 1 is authoritative as the *trigger*); the build itself may run asynchronously/queued (absorbing mechanism 2's batch nature) — but no manual Sheet step is the canonical path. The intranet records the handoff event and notifies Operations on merge; the loader builibreria in Moodle hidden; the instructor completes setup in Moodle; a final validation happens in Moodle before students see it. See PRD FR-33/FR-34/FR-37a.

**Still for architecture:** the detailed sequence/state machine, queue vs. synchronous build, error/retry, and idempotency (PRD FR-36). The *trigger* and *ownership boundaries* are no longer open; the *internal orchestration* is (tied to OQ-6 spike).

## C. Artifact generation pipeline (OQ-3)

Three candidate generation points appear in the docs for the course package / COURSE_MASTER_PLAN / MKT_BRIEFING:
- In the **wizard** ("we generate a `.md`"),
- At **approval** (platform PA),
- Inside the **loader** (Claude-API YAML generation).

Pin one source of truth per artifact. Note source typo: `COURSE_COURSE_MASTER_PLAN` → canonical `COURSE_MASTER_PLAN`.

## D. Platform track detail (PA / PB / PC / P101) — all undefined, on critical path (OQ-6)

- **P101** — Moodle user/role/enrolment via web services; assign Teacher/Editing Teacher, enrol instructor on publish; idempotency on re-run; uses `wstoken`. Open: which exact WS functions, role scoping, enrolment method.
- **PA** — Generate MKT_BRIEFING + COURSE_MASTER_PLAN (Markdown). Open: trigger point (see §C), exact schemas.
- **PB** — Temporal sequence/state machine: `approved` → generate → queue → batch build → `published` + Moodle-link write-back. Open: intermediate-state persistence, error/retry, write-back confirmation.
- **PC** — Loader consumes COURSE_MASTER_PLAN, builibreria course (hidden). Open: Plan → `PlanBCourseBuilder` schema mapping, write-back, idempotency, asset reference resolution, mid-build failure recovery.

**Recommendation carried from `docs/risks.md`:** spike the Moodle integration against the real loader codebase *first* — the riskiest, least-specified work is on the critical path.

## E. Marketing pipeline detail (deferred; stories 06b/07/08)

- 06b: customer.io is the email/lead layer (per user, 2026-06-16); the docs additionally describe **Metricool** for scheduled social drafts (X/LinkedIn) and **Canva** for branded assets, with **Nostr** as a manual-post exception (Metricool can't post Nostr). Posting-cadence proposal: max 1 new-course post/day, FIFO. These are deferred but should not be conflated with customer.io's transactional role.
- 07: instructor promo kit (horizontal + vertical Canva assets, 2–3 Spanish copy variants, Moodle link), emailed on `published`, independent of the `approved` email.
- 08: "entity with two views" pattern — same course/instructor data rendered as internal dashboard and public landing; featured flag; per-course Markdown export.

## F. Source provenance

- Consolidated working doc dated 2026-06-09; split into per-story files 2026-06-15; renumbered to "master" per recent commits. Content mixed ES/EN; EN normalization deferred to avoid churn on a moving spec.
- Jira label **ENG-271** is aspirational (see `docs/README.md` numbering note).
