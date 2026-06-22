---
story: "05"
title: Approval via GitHub PR — implementation tasks
status: draft
---

# Story 05 — Implementation tasks

> Tasks for [Story 05](../05-approval-github-pr.md). Owner: Ops.
> **Handoff mechanism — DECIDED (PRD OQ-1):** PR **merge triggers the build automatically**
> (the build itself may run async/queued); there is **no manual Sheet step**. Reconciliation
> with [Story 06](../06-handoff-moodle-publication.md) is resolved — see that file.
> **Still open:** the on-merge build depends on the loader/platform track (PC, OQ-6 spike); and
> GitHub↔canonical-identity must be reconciled (OQ-4, **build-blocking**) —
> [risks.md](../../risks.md#github-vs-idp-identity).

## Repo & structure

- [ ] Create the single `courses` repo.
- [ ] Define the per-course folder/Markdown structure (matching [Story 03](../03-course-submission.md)).
- [ ] **Companion `course.yml`** carries machine metadata (category, cohort, shortname, type) —
      DECIDED (architecture, resolves OQ-5). The reviewer writes the assigned Moodle category
      here at approval (FR-30a).

## Submission flow

- [ ] On submit, the **`course_bot` GitHub App** opens the PR in the `courses` repo with
      `COURSE_MASTER_PLAN.md` + `MKT_BRIEFING.md` + `course.yml` (architecture).
- [ ] Instructor authors/edits the curriculum as Markdown in the PR, then comments
      **`@course_bot finished_course`** → status advances to `submitted` (ChatOps trigger).
- [ ] GitHub auth / GPG-signed commits (nice-to-have). **GitHub PR author ↔ canonical identity
      (FR-6) blocked by OQ-4** — implemented in [Story 00](00-federated-identity.tasks.md).

## Review flow (Ops)

- [ ] Reviewer playbook: Approve / Request changes (comment required) / Close (comment required).
- [ ] Confirm push-to-PR re-opens review automatically; merge = sign-off.
- [ ] Notify the instructor on every review action via **customer.io** (FR-27/37), beyond
      GitHub's built-in PR notifications.
- [ ] **Rejected path (FR-24a):** on close-unmerged, notify the instructor with the reviewer's
      reason, keep the submission visible **read-only**; re-attempt requires a **new submission**
      (a rejected submission is not reopened in v1).

## Publish automation

- [ ] On merge, **automatically trigger** the handoff/build (no manual step); the build may run
      async/queued. Course is created in Moodle in **'Hidden from Students'** state.
- [ ] On trigger, the intranet **records the handoff event and notifies Operations** (FR-37a);
      detailed sequence/orchestration lives in [Story 06](../06-handoff-moodle-publication.md).

## Done when

- [ ] A merged PR is the sign-off and triggers the (single, agreed) Moodle upload mechanism.
