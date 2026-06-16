---
title: lds-intranet — Instructor Onboarding & Course Submission
status: draft
created: 2026-06-16
updated: 2026-06-16
epic: ENG-271
---

# PRD: lds-intranet — Instructor Onboarding & Course Submission

*Working title — confirm.*

## 0. Document Purpose

This PRD is for brería de Satoshi dev team. It defines **v1** of the intranet: the private workflow that takes a person from "I want to teach" through profile creation, course submission, Operations approval, and handoff of an approved course into Moodle. It is the source of truth for the **ENG-271** epic.

It is built on the existing `docs/` product spec (overview, roadmap, risks, 10 per-story files, and the PA/PB/PC/P101 platform track) — this PRD consolidates and reconciles that material rather than replacing it; the per-story files remain the detailed working notes. Vocabulary is anchored in the Glossary (§3) and must be used verbatim downstream. Features are grouped (§4) with globally-numbered functional requirements (FR-N) nested under them. Cross-cutting non-functional requirements live in §6, integrations in §7. Inferences I made are tagged `[ASSUMPTION]` inline and indexed in §10; genuinely undecided items are in §9 Open Questions. Technical "how" (tech stack, mechanism options, existing loader internals) is deliberately kept out of this PRD and lives in `addendum.md`.

## 1. Vision

LdS needs to grow its catalog — the immediate goal is onboarding **~50 new courses** within roughly **3 months**. Today there is no clean path for an instructor to propose a course and for Operations to capture everything they need; the process leans on Moodle, which was never meant to be the intake or approval surface.

The LdS intranet inverts that. It is a **standalone application that owns the course-submission lifecycle end to end** — from a person deciding to teach, through a guided wizard, drafts, and an Operations-led approval, to the moment an approved course is handed off and built in Moodle. Moodle stops being the container of the process and becomes its final destination: where published content lives and where students learn. After handoff, the intranet keeps the record as history — the foundation for future analytics.

This is the first slice of a larger intranet with **public and private fronts** connected. v1 delivers the private front (the workflow above). The public front (instructor profiles, course landing pages) and additional capabilities will come later, designed once the core lifecycle is proven. The intent is one smooth procedure that makes it easy for instructors to create a course while capturing, outside of Moodle, everything the Operations and Marketing teams need.

## 2. Target User

### 2.1 Jobs To Be Done

**Aspiring / existing instructor**
- Propose and submit a course without learning Moodle or fighting a generic form — one guided flow, save-and-resume.
- Present myself credibly (bio, links, experience) so LdS and students trust me.
- Know where my submission stands (draft → submitted → in review → approved → published) without chasing anyone.
- Once published, get what I need to promote my course to my own audience.

**Operations reviewer (LdS staff)**
- See a complete, consistent submission with everything needed to make an approve/reject call — no back-and-forth to collect missing pieces.
- Review, request changes, and approve in a tool the team already trusts (GitHub), with a durable audit trail.
- Trigger publication to Moodle on approval without manual data re-entry.

**Marketing (LdS staff)**
- Receive a marketing briefing as soon as a course is approved, so promotion work can start before the (slower) technical publish lands.

**LdS as an organization**
- Onboard ~50 courses in ~3 months with a repeatable, low-friction pipeline.
- Capture course + instructor data outside Moodle as the basis for future analytics and a future public front.

### 2.2 Non-Users (v1)

- **Students** — out of scope for the intranet in v1; they continue to use Moodle natively (including Nostr login, which stays Moodle-only). `[ASSUMPTION]` Student-facing surfaces are phase 2+.
- **The general public** — no public instructor profiles or course landing pages in v1 (deferred to phase 2).
- **Marketing as an active in-app operator** — marketing *receives* outputs (briefings) in v1 but the full promotion tooling (stories 06b/07/08) is deferred.

### 2.3 Key User Journeys

**UJ-1. Aspiring instructor submits a course, start to finish.**
- **Persona + context:** A Bitcoin developer with a following wants to teach a cohort-based course on LdS. New to LdS staff tooling; comfortable with GitHub.
- **Entry state:** Unauthenticated, on the LdS site; clicks **"Teach on LdS"**.
- **Path:** (1) Signs in via the intranet's identity provider — chooses GitHub (or Google, or a local account). (2) Lands in the **teacher profile wizard**, fills bio + links + experience. (3) Continues into **course submission**: course metadata (type, pitch, objectives, ideal student, ownership) then a per-class **content breakdown**. (4) Saves a draft partway through dinner, resumes the next day from the same step. (5) Submits — the intranet produces a review-ready package and opens it as a GitHub PR.
- **Climax:** A confirmation screen + email: "Your course is submitted and under review," with a link to track status.
- **Resolution:** Submission sits at `submitted` / in-review; instructor waits on Ops. **Edge case:** if Ops requests changes, the instructor edits and the same submission re-enters review (no fresh submission).

**UJ-2. Operations reviews and approves, triggering publication.**
- **Persona + context:** An LdS Operations reviewer responsible for course quality and catalog fit.
- **Entry state:** Authenticated as reviewer/Ops; notified a new submission PR is open.
- **Path:** (1) Opens the submission as a GitHub PR with the full course package. (2) Reviews; comments and **requests changes** where needed (a required comment). (3) Instructor revises; reviewer re-reviews. (4) **Approves** by merging the PR. (5) Merge initiates the handoff: documents are generated and the course is built in Moodle in a *hidden-from-students* state, with role/enrolment set for the instructor.
- **Climax:** The intranet records the **Moodle link** against the submission and flips its status to `published`.
- **Resolution:** Course live (hidden) in Moodle; instructor and marketing notified. **Edge case:** handoff is not instantaneous (batch) — status sits at `approved` until the loader confirms, and the UI communicates the delay.

**UJ-3. Instructor gets notified along the way (customer.io).**
- **Persona + context:** Same instructor as UJ-1, tracking progress passively via email.
- **Path:** Receives transactional emails at key transitions — profile/course submitted, changes requested, approved, and finally **published** (with the Moodle link and a promo kit). Their interest is also captured as a **lead** in customer.io the moment they start "Teach on LdS," even if they never finish.
- **Resolution:** Instructor stays informed without polling; LdS retains the lead for follow-up.

## 3. Glossary

*Downstream workflows and readers must use these terms verbatim. No synonyms.*

- **Intranet** — The standalone LdS application that owns the course-submission lifecycle and is the source of truth for intake, drafting, and approval. After handoff it retains the historical record.
- **Moodle** — The LMS where published courses live and students learn. In v1 it is the *destination* of the process, not its container.
- **Instructor** — A person who proposes/teaches a course. Becomes an instructor (intranet role) once they have ≥1 approved course.
- **Reviewer / Ops** — LdS staff who review submissions and approve or reject them.
- **Marketing** — LdS staff who promote courses; consume the MKT_BRIEFING.
- **Admin** — LdS staff with elevated configuration/management rights in the intranet.
- **Teacher profile** — The instructor's public-facing identity data (name/pseudo, bio, links, experience). Captured in the profile wizard.
- **Course submission** — A proposed course an instructor submits for approval: course **metadata** + per-class **content breakdown**.
- **Metadata** — Course-level fields: name, type (MOOC / cohort-based), pitch, objectives, ideal student, language, ownership, etc.
- **Content breakdown** — Per-class decomposition: class number/title, topics, objectives, activities, source materials.
- **Wizard** — The guided multi-step flow spanning entry → profile → course submission.
- **Draft** — An incomplete submission saved for later resumption. An intranet-owned entity, not a hidden Moodle course.
- **Submission status** — Lifecycle state of a submission: `draft` → `submitted` (in review) → `changes-requested` → `approved` → `published` (or `rejected`).
- **Approval** — Operations' sign-off, performed by merging the submission's GitHub PR.
- **`courses` repo** — The single GitHub repository where submissions live as Pull Requests.
- **Handoff** — The transition of an approved course from the intranet into Moodle: document generation → build in Moodle → write-back of the Moodle link.
- **COURSE_MASTER_PLAN** — The canonical content contract (a Markdown document) describing course structure and assets; consumed by the loader to build the course in Moodle.
- **MKT_BRIEFING** — A marketing-focused document derived from the submission; consumed by Marketing.
- **Loader** (`moodle-course-loader`) — The existing tool that consumes the COURSE_MASTER_PLAN and builds the course structure in Moodle via Moodle's web-service API.
- **Moodle link** — The URL of the built course in Moodle, written back to the intranet to mark a submission `published`.
- **Identity provider (IdP)** — The intranet's authentication service offering three login methods (local username/password, Google, GitHub), all resolving to a single canonical user identity.
- **customer.io** — The external service used for transactional/notification email and lead capture, integrated with the intranet.

## 4. Features

FRs are globally numbered and stable. **[v1]** = in the 3-month skeleton; **[later]** = deferred (kept here so dependencies are visible).

### 4.1 Identity & Access *(story 00)*

Full federated identity in v1 (2 developers dedicated). One canonical identity per person; three login methods.

- **FR-1** [v1] The intranet must authenticate users via an identity provider supporting three methods — local username/password, Google, and GitHub — with the user choosing the method.
- **FR-2** [v1] All three methods for the same person must resolve to a **single canonical user identity**; a user who later signs in via a different method must map to the same identity (account linking).
- **FR-3** [v1] On first successful sign-in, the intranet must create a user record and route the user into onboarding (profile wizard) or their dashboard as appropriate.
- **FR-4** [v1] The intranet must define and enforce its **own roles** — instructor, reviewer/Ops, marketing, admin — independent of Moodle's roles. (v1 exercises instructor + reviewer/Ops primarily.)
- **FR-5** [v1] Authentication must function independently of Moodle availability (intranet login does not depend on Moodle being up).
- **FR-6** [v1] The system must reconcile **GitHub identity** (used for PR authorship/approval in §4.5) with the canonical user identity, so PR actions map to the right intranet user. *(See OQ-4.)*
- **FR-7** [later] Migrate existing instructors to the canonical identity. Nostr remains a Moodle-only student login and is **not** part of the intranet IdP.

### 4.2 Entry Point & Teacher Profile *(stories 01, 02)*

- **FR-8** [v1] The site must present a discoverable **"Teach on LdS"** entry action in a consistent, documented location.
- **FR-9** [v1] The entry action must be **state-aware**: unauthenticated → sign-in (§4.1); authenticated with no submission → start a new draft; with existing draft(s) → submissions dashboard; established instructor → instructor dashboard.
- **FR-10** [v1] The profile wizard must capture required fields: name or pseudonym (public), email (pre-filled from identity), biography (50–300 words, editable later), GitHub username, whether they have an audience, and residency location (for payments).
- **FR-11** [v1] The profile wizard must capture optional fields: LinkedIn, Nostr, X, years of teaching experience, teaching environments (multi-select, shown only if experience given), and a free-text note.
- **FR-12** [v1] Profile data must be editable after initial submission.

### 4.3 Course Submission *(story 03)*

- **FR-13** [v1] The submission flow must capture course **metadata**: course name; course **type** (MOOC self-paced **or** cohort-based); language; elevator pitch; problem solved; 3 learning objectives; ideal-student description; content-ownership selection.
- **FR-14** [v1] For **cohort-based** courses, the flow must additionally capture: selection-process option, start date, end date, class count, class recurrence, time, and time zone. For **MOOC**, it must capture language selection. *(Conditional on type.)*
- **FR-15** [v1] The flow must capture a **content breakdown per class**: class number/title, topics, objectives, activities (quiz/exercises), and source materials.
- **FR-16** [v1] The flow must capture presentation video(s) for social media and the content-ownership / open-source licensing agreement.
- **FR-17** [v1] On submission, the system must produce a **review-ready package** (Markdown) structured for the GitHub PR workflow (§4.5), and this package must be the source for the MKT_BRIEFING and COURSE_MASTER_PLAN. *(Single point of generation — see OQ-3.)*

### 4.4 Drafts *(story 04)*

- **FR-18** [v1] The system must let a user **save a draft** and **resume** from the last incomplete step, with prior steps pre-filled, across sessions. Drafts are intranet-owned entities.
- **FR-19** [v1] A draft must not enter review until **explicitly submitted**.
- **FR-20** [v1] The dashboard must list a user's draft(s) with status and a "Continue" action.
- **FR-21** [later] **Multiple simultaneous drafts** per user and fine-grained per-step autosave. *(v1 supports at least single-draft save/resume; multi-draft deferred.)* `[ASSUMPTION]`
- **FR-22** [later] Preserve abandoned-draft state for funnel/abandonment analytics.

### 4.5 Approval via GitHub PR *(story 05)*

- **FR-23** [v1] All submissions must live as Pull Requests in a single **`courses` repo**; submitting a course opens a PR containing the course package as Markdown.
- **FR-24** [v1] Reviewer/Ops must be able to **approve** (merge), **request changes** (with a required comment), or **reject** (close) a submission via the PR.
- **FR-25** [v1] Pushing revisions to an existing submission must re-enter review on the **same** submission (no new submission created).
- **FR-26** [v1] Submission status in the intranet must stay in sync with the PR state (open / changes-requested / approved-merged / closed).
- **FR-27** [v1] Every review action must notify the instructor (via §4.7). Reviewers must comment when requesting changes or rejecting.
- **FR-28** [v1] Category / cohort / shortname metadata needed for the Moodle build must travel with the submission (companion file or in the package — see OQ-5).

### 4.6 Handoff to Moodle & Publication *(story 06; platform PA/PB/PC/P101)*

The critical path. Publishing a live course in Moodle is the only step that delivers real end value.

- **FR-29** [v1] On approval, the system must generate the **COURSE_MASTER_PLAN** (loader input) and **MKT_BRIEFING** (marketing input) from the submission package.
- **FR-30** [v1] The system must hand the COURSE_MASTER_PLAN to the **loader**, which builds the course structure in Moodle in a **"hidden from students"** state.
- **FR-31** [v1] On publish, the instructor must be assigned the appropriate **Teacher role and enrolment** in the Moodle course (platform P101).
- **FR-32** [v1] On successful build, the **Moodle link** must be written back to the intranet and the submission status set to `published`.
- **FR-33** [v1] The handoff must be defined as a single, authoritative **sequence/state machine** (`approved` → generated → built → `published`), reconciling the two mechanisms described in the source docs (PR-merge automation vs. batch loader). *(See OQ-1.)*
- **FR-34** [v1] The UI must communicate that handoff is **not instantaneous** (batch), so `approved` may persist before `published`.
- **FR-35** [v1] A person with ≥1 approved course must be marked **instructor** in the intranet.
- **FR-36** [v1] The handoff must define behavior on **failure** (build error / retry) and **re-run** (idempotency) so a failed or repeated handoff does not corrupt Moodle state. `[ASSUMPTION]` (v1 may handle this manually but the behavior must be specified.)

### 4.7 Notifications & Lead Capture *(customer.io)*

- **FR-37** [v1] The intranet must send transactional emails via **customer.io** at key transitions: profile/course submitted, changes requested, approved, and published.
- **FR-38** [v1] When a user starts **"Teach on LdS,"** the intranet must register them as a **lead** in customer.io, even if they never complete the wizard.
- **FR-39** [v1] The **published** notification to the instructor must include the **Moodle link**. (Promo-kit contents are §4.8 / deferred.)

### 4.8 Marketing & Promotion *(stories 06b, 07, 08 — deferred)*

- **FR-40** [later] On `approved`, notify Marketing with the MKT_BRIEFING so promotion drafting can begin before publish.
- **FR-41** [later] On `published`, send the instructor a **promotion kit** (branded assets + copy variants + Moodle link).
- **FR-42** [later] Automated promotion scheduling pipeline (Metricool/Canva), incl. the Nostr manual-post exception and posting-cadence rules.
- **FR-43** [later] **Public instructor profile** and per-course public landing pages; featured-course flagging; course-data/Markdown exports for marketing.

## 5. Success Metrics

| # | Metric | Why it matters | Target (rough) |
|---|---|---|---|
| SM-1 | Courses onboarded **end-to-end** (submitted → published) | The whole point of v1 | Trending toward **50** in ~3 months `[ASSUMPTION on exact pacing]` |
| SM-2 | **Instructor completion rate** (started wizard → submitted) | Measures wizard friction | Establish baseline, then improve |
| SM-3 | **Time** from "Teach on LdS" → submitted | Measures flow smoothness | Establish baseline |
| SM-4 | **Ops time-to-approval** (submitted → approved/rejected) | Measures review throughput | Establish baseline |

**Counter-metrics** (guard against gaming the above):
- **CM-1** Rejection / rework rate after approval (don't approve junk to hit SM-1).
- **CM-2** Handoff failure rate (don't mark `published` without a working Moodle course).
- **CM-3** Drafts abandoned mid-wizard (a fast time-to-submit means nothing if most never submit).

## 6. Non-Functional Requirements

- **NFR-1 Availability independence:** Intranet auth and the submission workflow must function when Moodle is down (degrading only the handoff step).
- **NFR-2 Data ownership & history:** The intranet is the source of truth for intake/approval data and must retain the record post-handoff as the analytics foundation.
- **NFR-3 Auditability:** Approval history must be durable and traceable (PR trail in the `courses` repo; status transitions recorded in the intranet).
- **NFR-4 Identity integrity:** Exactly one canonical identity per person across the three login methods; no duplicate or orphaned accounts.
- **NFR-5 Idempotent handoff:** Re-running a handoff must not create duplicate Moodle courses or corrupt enrolment.
- **NFR-6 Language:** Product UI/spec normalized to **English**; instructor-authored content may be ES/EN. `[ASSUMPTION]`
- **NFR-7 Security:** Standard auth hygiene for local accounts (password storage, session handling) and least-privilege role enforcement across the four intranet roles.

## 7. Integrations

- **Moodle** — Destination LMS. Integration via the loader / Moodle web-service API: create course (hidden), build structure, upload assets, assign Teacher role + enrolment, return course URL. (Internals → addendum.)
- **GitHub** — The `courses` repo hosts submissions as PRs; merge = approval; PR state mirrors submission status. Also a login method (§4.1) and the identity to reconcile (FR-6).
- **customer.io** — Transactional email + lead capture (§4.7).
- **Marketing tooling (Metricool / Canva)** — Deferred (§4.8); noted so it isn't confused with customer.io's role.

## 8. Out of Scope (v1) / Future

- Public front: instructor profiles, course landing pages, featured courses, exports (story 08).
- Full marketing-promotion automation and instructor promo kits (stories 06b, 07, owner Aria).
- Teacher help/reference documentation (story 09 — stub, owner: Ivan Fuentes).
- Multi-draft + fine-grained autosave (FR-21).
- Student-facing intranet surfaces; Nostr in the intranet IdP.
- Post-publish analytics pipeline (data is *retained* in v1; analytics built later).

## 9. Open Questions

These are genuine decisions, several of them "one-way doors." They should be resolved during architecture/solutioning (the platform track especially) — **not** silently assumed.

- **OQ-1 Handoff mechanism (one-way door).** The docs describe two conflicting mechanisms: PR-merge → automated action → Moodle, vs. COURSE_MASTER_PLAN → sheet → **batch** loader. FR-33 requires a single authoritative sequence. Which is it? *(Highest-impact technical decision; shapes all of §4.6.)*
- **OQ-2 Post-publish data consistency (one-way door).** Are edits allowed after publication, and on which side (intranet vs. Moodle)? Determines whether handoff is fire-and-forget or requires bidirectional reconciliation. *(Was the source doc's original opening question.)*
- **OQ-3 Artifact generation point.** Where is the course package / COURSE_MASTER_PLAN / MKT_BRIEFING generated — in the wizard, at approval, or inside the loader? FR-17/FR-29 assume a single point; pin it.
- **OQ-4 GitHub ↔ canonical identity mapping.** With full federation in v1 (not GitHub-only), PR authorship is GitHub-based but the canonical identity may have signed in via Google/local. How are they linked (FR-6)? GPG-signed commits sharpen this.
- **OQ-5 Submission metadata format.** Category/cohort/shortname as a companion `.yml` or inline in the Markdown package (FR-28)?
- **OQ-6 Platform track is undefined.** PA, PB, PC, P101 are all unspecified yet sit on the critical path. **Spike the Moodle integration against the real `moodle-course-loader` before committing dates.** *(Carry into architecture + technical-research.)*
- **OQ-7 Confirmation-email field** in the profile wizard — keep or drop (FR-10/11)? Currently ambiguous in the docs.

## 10. Assumptions Index

- `[ASSUMPTION]` v1 supports single-draft save/resume; multi-draft deferred (FR-21).
- `[ASSUMPTION]` Handoff failure/idempotency behavior must be specified even if handled manually in v1 (FR-36).
- `[ASSUMPTION]` Students and the public front are phase 2+ (§2.2, §8).
- `[ASSUMPTION]` UI language normalized to English; instructor content may be ES/EN (NFR-6).
- `[ASSUMPTION]` 50 courses is a directional target over ~3 months, not a hard SLA (SM-1).
