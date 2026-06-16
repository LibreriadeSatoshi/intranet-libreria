---
id: "05"
title: Approval via GitHub PR
track: product
status: draft
mvp: true
owner: Ops
depends_on: ["03"]
risk: high
jira: ENG-276
---

# Story 05 — Course submissions reviewed via GitHub

**Tasks:** [implementation breakdown →](tasks/05-approval-github-pr.tasks.md)

## User story

**As** an instructor, **I want** to submit the final version of a course and get clear
feedback, **so that** I know whether it's approved or needs changes. Additionally, **I want**
the course to be automatically uploaded to Librería's Moodle.

## Why GitHub

Teachers already know (or should know) Markdown and GitHub, and both are well-known in the
Bitcoin ecosystem. Instead of building a custom approval workflow, we reuse GitHub's Pull
Request review process. Courses are written as Markdown files following the structure from
[Story 03](03-course-submission.md).

## Where

A single GitHub repository for courses (the `courses` repo). All submissions are Pull
Requests against this repo. Each course lives in its own Markdown files/folder.

## How it works

1. **Submit** — The instructor forks/branches the `courses` repo and opens a PR adding their
   course as Markdown files.
2. **Review** — A reviewer from **Ops** reviews the PR:
   1. **Approve** the PR → the course is accepted.
   2. **Request changes** → the instructor edits and pushes again.
   3. **Close** the PR → the submission is rejected.
3. **Changes loop** — Pushing new commits to the same PR re-opens it for review
   automatically. No new submission needed.
4. **Accepted** — Ops **merging** the PR is the sign-off.
5. **Publish** — On merge, a GitHub Action automatically uploads the course to Moodle in the
   **'Hidden from Students'** state. No manual step needed.

## Acceptance criteria

- A course submission is a Pull Request against the `courses` repo, containing Markdown files
  in the required structure.
- Status is what GitHub already shows: open, changes requested, approved, merged, or closed.
- Reviewers must leave a comment when requesting changes or closing.
- Every review action notifies the instructor (GitHub's built-in notifications).
- A merged PR triggers a GitHub Action that uploads the course to Moodle.

## Notes

- Category, cohort, and short name can be defined as a companion `.yml` file, or in the
  Markdown itself (**TBD**).
- Authentication for PRs should be provided by GitHub.
- GPG-signed commits would be a **very** nice addition, in sync with how the Bitcoin
  ecosystem works.

## Open questions

- **Companion `.yml` vs Markdown** for category/cohort/shortname — decide (PRD OQ-5, → architecture).
- **Identity reconciliation:** PR authorship/auth is GitHub-based, but the rest of the system
  keys on the IdP `sub` ([Story 00](00-federated-identity.md)). How do GitHub identity and
  IdP identity map? GPG signing sharpens this question (PRD OQ-4, build-blocking). See
  [risks.md](../risks.md#github-vs-idp-identity).

## Resolved

- **Handoff mechanism** — RESOLVED (PRD OQ-1): the PR merge **triggers the build automatically**
  (build may run async/queued); there is **no manual Sheet step**. Unified with
  [Story 06](06-handoff-moodle-publication.md).
