# PRD Quality Review — libreria-intranet (2026-06-16)

Scope: `prd.md` + `addendum.md`. Audience: ~3-dev team, ~3-month v1 build. Findings only; no praise.

---

## Verdict

A solid, well-organized PRD with clear vision and good traceability, but **not yet buildable for the critical path**. The handoff/platform track (§4.6, PA/PB/PC/P101) is openly undefined while sitting on the only step that delivers end value, and the status lifecycle is inconsistent across §2.3, §4, and §9. Several FRs that are tagged `[v1]` are effectively underspecified. Coverage gaps in access control, asset upload, and rejection handling will force the team to guess.

---

## CRITICAL

**C-1. Critical-path work (§4.6 / OQ-6) is explicitly undefined.**
- Touches: FR-29–FR-36a, addendum §D, OQ-6.
- The PRD admits PA/PB/PC/P101 are "all undefined" and on the critical path. The handoff mechanism (OQ-1) is marked RESOLVED in §9 but the addendum §B still describes two contradictory architectures (GitHub Action instantaneous vs. Sheet→batch loader async) that "imply different latency, error-handling, and state-machine designs." A team cannot build v1's only value-delivering step from this.
- Fix: Run the recommended loader spike *before* committing the PRD as build-ready. Pick one authoritative handoff path and fold the resulting sequence/state machine into FR-33 (don't leave it at "to be designed in architecture"). Demote OQ-1 from RESOLVED to "mechanism chosen, sequence pending" — it is not resolved.

**C-2. Status lifecycle is used inconsistently across the document.**
- Touches: §2.3, FR-26, Glossary §3, OQ status states.
- Glossary defines: `draft → submitted → changes-requested → approved → published (or rejected)`. But:
  - §2.3 UJ-1 path uses "draft → submitted → in review → approved → published" — introduces "in review" which is not a status; §2.1 JTBD repeats it.
  - §2.3 UJ-1 resolution says submission sits at "`submitted` / in-review" — conflates two labels for one state.
  - FR-26 enumerates PR-mirrored states as "open / changes-requested / approved-merged / closed" — `approved-merged` and `closed` are not glossary statuses, and `closed` maps ambiguously to either `rejected` or `published`.
  - `changes-requested` appears in the glossary lifecycle but no FR drives the transition *back* to `submitted` after revision (FR-25 says "re-enter review" without naming the target state).
- Fix: Define one canonical state set and a transition table (from → event → to). Map PR states to it explicitly (open=submitted; review-comment-changes=changes-requested; merged=approved; closed-unmerged=rejected). Replace all "in review"/"approved-merged"/"closed" prose with glossary terms verbatim (§3 mandates verbatim use).

**C-3. Access control across the 4 roles is undefined.**
- Touches: FR-4 (defines roles but not permissions), NFR-7 (mentions "least-privilege" with no matrix).
- The PRD names instructor/reviewer/Ops/marketing/admin but never states who can see/do what: who reads a submission, who sees other instructors' drafts, who can approve, what marketing can access pre-publish, what admin configures. "Least-privilege enforcement" is asserted without the rules to enforce.
- Fix: Add a role × capability matrix (view/edit/submit/review/approve/reject/configure across submission, draft, profile, Moodle-link). This is a v1 requirement, not architecture detail.

---

## HIGH

**H-1. No acceptance criteria anywhere; most FRs are untestable as written.**
- Touches: all FRs.
- FRs state intent ("must capture", "must notify", "must stay in sync") but none has measurable pass/fail conditions. E.g. FR-26 "stay in sync" — within what latency? on which PR events? FR-18 "resume from last incomplete step" — step granularity? FR-32 write-back — on what build signal?
- Fix: Add Given/When/Then or bulleted AC to each `[v1]` FR. Prioritize FR-17, FR-18, FR-23–28, FR-29–36.

**H-2. Rejection path is a dead end — undefined.**
- Touches: FR-24 (reject = close PR), §2.3 (no rejection journey), §8.
- `rejected` is a terminal status but the PRD never says what happens to a rejected submission: Can the instructor resubmit? Is it a new submission or the same one (contrast FR-25 for changes-requested)? Is the lead/notification handled? Is the PR reopenable? UJ-1 only covers the changes-requested edge case, not rejection.
- Fix: Add an FR (and a short journey or edge case) for rejection: notification, instructor next-step, whether resubmission creates a new entity, data retention.

**H-3. File / asset upload is required by features but has zero specification.**
- Touches: FR-15 (source materials), FR-16 (presentation videos), §7 Moodle ("upload assets").
- The wizard captures "source materials" and "presentation video(s)" and the loader "uploads assets," but there is no requirement on: allowed file types, size limits, count limits, where assets are stored pre-handoff (intranet? GitHub LFS? external?), how they travel through the GitHub-Markdown read-only phase (FR-17a) into Moodle, or video hosting (embedded vs. linked — addendum mentions "embeds video" for the loader but nothing on intake).
- Fix: Add FR(s) for asset intake: types, max size/count, storage location across the wizard→GitHub→Moodle lifecycle, and how binary assets survive FR-17a's "content becomes read-only in GitHub Markdown."

**H-4. Analytics foundation: NFR-2 promises a data foundation but no FR specifies what to capture.**
- Touches: NFR-2, FR-22 [later], SM-1–4, CM-1–3, §8.
- The PRD repeatedly claims the intranet is "the data foundation for future analytics" and lists metrics (completion rate, time-to-submit, time-to-approval, abandonment) — but no v1 FR requires capturing the events/timestamps those metrics need. FR-22 (abandonment state) is tagged `[later]`, which directly undercuts CM-3 and SM-2/SM-3 baselines. If timestamps aren't recorded from day 1, the baselines in §5 are unobtainable.
- Fix: Add a v1 FR: record status-transition timestamps and wizard-step progression events (including abandonment) from launch, even if dashboards are deferred. Move the *capture* part of FR-22 into v1; keep only the *reporting* deferred.

**H-5. FR-2a (existing-student-becomes-teacher) and FR-6 (GitHub↔canonical identity) are `[v1]` but their mechanism is in OQ-4 (unresolved).**
- Touches: FR-2a, FR-6, OQ-4.
- These are tagged v1 and federation is "full in v1," yet the linking/fallback strategy and PR-author reconciliation are explicitly deferred to architecture. PR approval (FR-24) can't reliably map merges to intranet users without this. This is a v1 blocker masquerading as an open question.
- Fix: Either resolve OQ-4 with a concrete linking rule before build, or descope full federation for v1 (e.g., GitHub-first) — the PRD argues against the pilot but the unresolved identity mapping is exactly the risk a pilot would retire.

---

## MEDIUM

**M-1. "Single point of generation" for artifacts is assumed by FRs but unresolved (OQ-3).**
- Touches: FR-17, FR-29, OQ-3, addendum §C. FR-17 and FR-29 both assume a single generation point but three candidates exist (wizard / approval / loader). Affects what the GitHub package contains. Fix: pin per-artifact source before building §4.3/§4.6.

**M-2. Submission metadata format (FR-28) unresolved but blocks the Moodle build.**
- Touches: FR-28, FR-30a, OQ-5. Category/cohort/shortname "must travel with the submission" but format is open. Reviewer category selection (FR-30a) depends on it. Low stakes per PRD, but it gates the build. Fix: decide companion `.yml` vs inline; specify the field set.

**M-3. Notification matrix is incomplete and partly contradictory.**
- Touches: FR-27, FR-37, FR-37a, FR-39, FR-40. FR-27 says "every review action must notify the instructor"; FR-37 lists only submitted/changes-requested/approved/published — "rejected" is missing from the transactional list despite being a review action. Marketing notification on `approved` is FR-40 `[later]`, but §2.1 and §2.3 UJ-3 present marketing-on-approval as a v1 JTBD ("before the slower technical publish lands"). Fix: add `rejected` to FR-37; reconcile whether marketing-briefing-on-approval is v1 or deferred (the JTBD and §4.8 disagree).

**M-4. FR-21 / FR-18 single- vs multi-draft boundary is fuzzy.**
- Touches: FR-18, FR-20 ("list a user's draft(s)"), FR-21. FR-21 defers multi-draft, but FR-20 says the dashboard lists "draft(s)" (plural) and FR-9 routes on "existing draft(s)." If v1 is single-draft, the dashboard/routing language should say so. Fix: make singular/plural consistent with the `[ASSUMPTION]` of single-draft v1.

**M-5. Confirmation-email field (FR-10) has no defined behavior.**
- Touches: FR-10, OQ-7 (RESOLVED only as "required"). It's required but the PRD never says what it does — match against the primary email? block submission on mismatch? trigger a verification step? Resolving "is it required" left "what does it do" open. Fix: specify the validation/verification behavior.

**M-6. Biography 50–300 words but other fielibreria have no constraints.**
- Touches: FR-10, FR-11, FR-13 ("3 learning objectives"). Bio has a word range and objectives are fixed at 3, but pitch/problem/ideal-student/topics have no length or required/optional bounds beyond the required/optional split. For a wizard with a measured completion rate (SM-2), field-level validation rules matter. Fix: add min/max/required constraints per field, or state they're intentionally free-form.

---

## LOW

**L-1. Working title unconfirmed.** Header line 11 "*Working title — confirm.*" and §0 epic ENG-271 noted as "aspirational" in addendum §F. Resolve before this is the source of truth.

**L-2. `customer.io` lead capture (FR-38) lacks dedupe/PII note.** Lead created "the moment they start." No mention of dedup against existing identity or what data is sent. Minor, but a privacy/consent note belongs here.

**L-3. NFR-6 language scope vs. content.** UI normalized to English, content ES/EN. The GitHub-Markdown package and generated COURSE_MASTER_PLAN/MKT_BRIEFING language is unstated — relevant since marketing copy variants (addendum §E) are Spanish. Clarify artifact language.

**L-4. SM targets are mostly "establish baseline."** Acceptable for a first build, but SM-2/3/4 have no even-rough target, so "success" is undefined for 3 of 4 metrics. Consider a directional target once H-4's capture lands.

**L-5. Minor padding.** §0 Document Purpose and §1 Vision restate the Moodle-inversion thesis three times (the BEFORE/NOW diagram, the prose above and below it). Tighten — it's the strongest idea in the doc and doesn't need three passes.

---

## Summary of where the build will stall first

1. Handoff/platform track undefined (C-1) — can't build the value step.
2. Status lifecycle inconsistency (C-2) — every state-driven FR inherits the ambiguity.
3. No access-control matrix (C-3) and no acceptance criteria (H-1) — pervasive guessing.
4. Rejection (H-2), asset upload (H-3), analytics capture (H-4), identity mapping (H-5) — concrete gaps that surface mid-build.
