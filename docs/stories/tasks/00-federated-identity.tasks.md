---
story: "00"
title: Federated identity — implementation tasks
status: draft
---

# Story 00 — Implementation tasks

> Tasks for [Story 00](../00-federated-identity.md).
> **Identity scope — DECIDED (PRD 2026-06-16):** full 3-method federation in v1 (local
> username/password + Google OIDC + GitHub OIDC via self-hosted Authentik); 2 devs dedicated.
> The GitHub-OIDC-only pilot path is **not** taken. Tasks below implement the full IdP.
> Still open (→ architecture, OQ-4, **build-blocking**): GitHub↔canonical-identity mapping and
> the existing-student→teacher linking fallback.

## Spike (do first)

- [ ] Validate Authentik for the teachers/staff volume (deployment size, cost).
- [ ] Connect Moodle 5.1 as an OAuth2 **core** client (config, not plugin); confirm the
      `/public` plugin move after upgrade doesn't break it.
- [ ] Confirm a **stable 1:1 `sub`** across local / Google / GitHub on the same IdP account.
- [ ] Decide the student→teacher linking strategy + fallback when the identifier doesn't match.

## IdP setup

- [ ] Stand up Authentik (self-hosted) with the deployment/runbook.
- [ ] Configure Google OIDC provider.
- [ ] Configure GitHub OIDC provider.
- [ ] Enable local username/password accounts.

## Intranet integration

- [ ] Add the intranet as an OIDC client of the IdP.
- [ ] Persist only the IdP **`sub`** reference on the intranet user (not Google/GitHub sub).
- [ ] First-login provisioning: create instructor/staff profile referenced by `sub`.
- [ ] Failure-independence check: intranet auth works with Moodle down.

## Moodle integration

- [ ] Configure Moodle 5.1 as an OAuth2 client of the IdP.
- [ ] Keep `wstoken` for Web Service calls (separate plane from the login token).

## Linking & migration

- [ ] Implement existing-student → teacher linking (auto-link on matching identifier).
- [ ] Build the manual linking step + fallback for the non-matching case.
- [ ] One-off migration: map existing teachers/staff to their IdP `sub`.

## Done when

- [ ] A new user logs in via the landing → IdP authenticates → profile auto-created by `sub`.
- [ ] The three methods resolve to one account/`sub`; Moodle-down does not block login.
