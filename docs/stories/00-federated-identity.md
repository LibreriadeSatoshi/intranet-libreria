---
id: "00"
title: Federated identity via dedicated IdP
track: product
status: draft
mvp: true
owner: TBD
depends_on: []
risk: high
jira: ENG-272
---

# Story 00 — Federated identity via dedicated IdP

**Tasks:** [implementation breakdown →](tasks/00-federated-identity.tasks.md)

> *IdP = Identity Provider · OIDC = OpenID Connect (identity layer on top of OAuth 2.0) ·
> `sub` = user identifier, unique per IdP.*

## User story

**As** an aspiring teacher / staff member, **I want** to log into the intranet with a single
account, **so that** I don't manage separate credentials and my work isn't interrupted if
Moodle is down.

## Model (proposed)

- Dedicated IdP that Google and GitHub connect to. Intranet and Moodle are independent 
  clients of the IdP via OIDC.
- The IdP is the single source of identity for anyone entering through the intranet. Authentication
  (who you are) lives in the IdP; authorization (what you can do) lives in each system 
  separately.

## Acceptance criteria

- **Login methods (via IdP):** user/password (local account in the IdP), Google
  (OIDC) or GitHub (OIDC); the user chooses. Nostr is not an intranet method; it lives only in
  Moodle (native). A teacher using Nostr in Moodle accesses the intranet with any of
  the three IdP methods, without bringing an external social identity.
- **Unique identity:** a single record (the IdP account); the intranet only stores the
  reference to the **IdP `sub`** (not the Google or GitHub `sub`). The three methods
  coexist in the same IdP account and resolve to the same `sub`.
- **Failure independence:** if Moodle is down, the IdP and the intranet continue authenticating.
  Teacher/staff identity does not depend on Moodle.
- **Provisioning on first login:** a new user enters through the landing page → IdP authenticates → the
  intranet creates their profile (instructor/staff) referenced by the IdP `sub`.

## Entry gates (principle: the gate decides the method)

- Entry through the intranet (landing page "Become a teacher", or staff) → IdP (Google/GitHub).
- Direct entry through Moodle (student to their course) → Moodle native login
  (Google/GitHub/Nostr/user-pass). Without touching the IdP.

## Single friction case — existing student → teacher

Occurs on the landing page. The user already has a Moodle account (created as a student). When entering
through the intranet via IdP, if the identifier matches (same Google/GitHub), it is linked. If not,
there is a step to link their existing Moodle account to the IdP identity. Define the
linking strategy and fallback.

## Scope — students out of the IdP for now

Students remain in native Moodle. The IdP is only for teachers/staff. Expanding to
students is a future phase (adding users to the same IdP, without redoing the architecture).

## Authorization (NOT managed by the IdP)

- The IdP provides **identity and the broad role** (staff/instructor). Fine-grained authorization is
  managed by each system.
- Intranet roles (own): instructor, reviewer/Ops, marketing.
- Moodle roles (own, managed in Moodle): Student, Teacher, Manager, admin. The
  loader assigns Teacher upon publishing ([P101](../platform/P101-moodle-user-role-enrolment.md));
  Manager and admin are managed in Moodle. The Moodle admin role is orthogonal to the intranet.

## Technical notes

- IdP: **Authentik self-hosted** (proportioned to the limited volume of teachers/staff). Only
  Google/GitHub (standard OAuth, out of the box); no Nostr-stage development, as Nostr remains
  outside the IdP.
- The IdP OIDC token is for login. Calls to Moodle Web Services (create courses,
  assign roles, [P101](../platform/P101-moodle-user-role-enrolment.md)) continue using
  `wstoken`. Authentication and API integration are separate planes.
- Moodle 5.1: connect Moodle to the IdP as a core OAuth2 client (config, not plugin). 5.1 requires
  moving plugins to `/public` after upgrade.
- Migration: map existing teachers/staff to the IdP `sub` (one-off, not permanent logic).

## Recommended spike (before implementing)

Validate Authentik for the teacher/staff volume, connect Moodle 5.1 as an OIDC client,
confirm stable 1:1 `sub`, and the student→teacher linking strategy. (It is
simplified compared to previous versions: no Nostr piece.)

## Open questions

- **Student→teacher linking strategy** when the identifier does NOT match — define
  the flow and the fallback (PRD OQ-4, build-blocking, → architecture).

## Resolved

- **Full Authentik in MVP?** — RESOLVED (PRD, decision-log Q1): going for **complete federation
  of 3 methods in v1** (local + Google + GitHub via Authentik self-hosted), with 2 dedicated devs.
  Not taking the GitHub-OIDC-only pilot.
