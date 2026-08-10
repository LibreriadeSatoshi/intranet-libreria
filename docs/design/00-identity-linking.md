---
title: Identity linking design (Gap G2)
status: draft
jira: ENG-326
story: "00"
covers: [FR-2, FR-2a, FR-6, OQ-4]
inputs: [ENG-325 spike (spikes/authentik/README.md)]
---

# Identity linking — design (G2)

Detail design for the three linking flows left open by architecture (G2 / OQ-4).
Once approved, the Linking & migration tasks of ENG-295 (ENG-330) are implementable.

**Already decided (architecture, not revisited here):** one canonical identity per person
= one Authentik account; the intranet persists only the Authentik `sub`; Moodle WS calls
keep using `wstoken`. **Spike facts this design relies on (ENG-325):** the `sub` is stable
1:1 across local/Google/GitHub on the same account; `email_link` auto-links a social login
to an existing account when the verified email matches, otherwise enrollment creates a new
account (type *external*); users self-link additional sources in Settings → Connected
services; a linked GitHub source stores the **numeric GitHub user ID** as `identifier`.

## Invariant

> Method-to-identity linking lives **in Authentik** (source connections). The intranet never
> stores Google/GitHub identifiers as identity — it stores the `sub` and, where useful,
> read-through caches of Authentik data. Auth.js `Account` rows only ever reference the
> Authentik provider (the intranet never talks to Google/GitHub OAuth directly), so they
> cannot be used for per-method linking — do not design against them.
>
> **Authentik stays stock.** Everything this design needs from the IdP is configuration
> (flows, stages, expression policies, sources, scope mappings) — no plugins, no forks, no
> custom services. That configuration is captured as versioned **Authentik blueprints**
> (YAML) in the repo and applied at deploy time (ENG-327), never hand-clicked. All actual
> code in this design lives in the intranet (T3 app).
>
> **The intranet↔Authentik surface is OIDC only.** Login uses the standard endpoints
> (authorize/token/jwks) via Auth.js, and everything else the intranet needs from Authentik
> travels **inside the id_token as claims** (see Flow 3). No Authentik REST API calls at
> runtime, no service token to custody.

## Flow 1 — email-match linking (FR-2)

**Trigger:** a user signs in via a source (Google/GitHub) and an Authentik account with the
same **verified** email already exists.

**Design:** keep `user_matching_mode: email_link` on both sources — the account is linked
and the user lands in it (spike-verified). The PRD's "explicit confirmation step" is
implemented as a **customized source-authentication flow** in Authentik: a prompt stage
shown only when the matching path fires — "An account `<username>` with this email already
exists. Link this <source> login to it?" Confirm → link (standard behaviour); deny → abort
with a support contact (no silent second account).

- Both Google and GitHub only expose verified emails through these sources, so the match
  signal is trustworthy; the confirmation is UX safety, not a security control.
- Compensating control either way: Authentik event notification (email) on
  `source_linked`, so the account owner learns a new method was attached.
- **De-scope option (call it in review):** if the flow customization proves >1 day of work,
  v1 ships plain `email_link` + the notification email, and the prompt stage becomes a
  fast-follow. The risk accepted is a user linking without realizing they had an account.

## Flow 2 — existing student → teacher (FR-2a)

This flow links a **Moodle account** to the canonical identity — it is Moodle↔`sub`, not
method↔method (that is Flow 1's job).

**Trigger:** during intranet onboarding (first login, profile wizard), the person may
already have a Moodle student account.

**Design:**

1. Wizard step: "Do you already have an account on the Librería campus?" (skippable).
2. **Auto-match:** look up Moodle by the canonical email (`core_user_get_users` via
   `wstoken`). Exactly one match → show masked confirmation ("`i***@l***.com`, is this
   you?") → store `moodle_user_id` on the intranet profile (keyed by `sub`). This is the
   FR-2a "matching identifier" happy path.
3. **Fallback (non-matching):** the user enters their Moodle username or email → the
   intranet sends a 6-digit code to **the email registered in Moodle** (not the address the
   user typed claims to own) → correct code proves control of the Moodle account → link.
4. **Residual fallback:** no email on the Moodle account (e.g. Nostr-only students) or
   verification impossible → the claim lands in an **Ops review queue**; Ops confirms
   out-of-band and links manually. Small volume makes this acceptable.
   **Queue owner (decided 2026-08-10):** members of the Authentik **admins group** — the
   `groups` claim ships in the default `profile` scope (spike-verified: akadmin's token
   carries `groups: ["authentik Admins"]`), so the intranet maps that group to its Ops
   role from the token alone, OIDC-only. If IdP administration and course-ops ever need
   to diverge, introduce a dedicated Authentik group and map that instead — same
   mechanism, no code change beyond the group name.
5. No claim → new/no `moodle_user_id`; the loader (P101) creates or links the Moodle user
   at first publication. A later claim goes through the same steps from profile settings.

**Data:** `Profile.moodleUserId: Int?` + `moodleLinkVerifiedAt` + audit event
(`identity.moodle_linked`, actor, method: auto|email-code|ops). Duplicate protection: a
`moodle_user_id` may be linked to at most one `sub` (unique constraint); a second claim on
a taken account goes to the Ops queue instead of failing silently.

## Flow 3 — GitHub PR author ↔ canonical identity (FR-6)

**Decision: Authentik is the mapping's source of truth, delivered to the intranet as an
id_token claim (Option A, OIDC-only variant).**

- **Transport:** a custom **scope mapping** on the OIDC provider (config → blueprint) adds
  a `github_id` claim to the id_token, read from the user's GitHub source connection
  (`identifier` = GitHub's numeric user ID, spike-verified, e.g. `5514150`). No claim when
  GitHub is not linked.
- **Requirement gate:** to submit a course (any flow that ends in a PR), the session must
  carry `github_id`. If missing, the wizard shows "Connect GitHub" — a deep link to
  Authentik Settings → Connected services, where self-service linking already exists
  (spike-verified) — followed by a **silent re-auth** (redirect with the live IdP session)
  so the fresh token picks the claim up. Linking happens via real GitHub OAuth, so the
  identity is **verified**, not self-declared.
- **Reconciliation:** GitHub webhooks deliver the PR author's numeric user ID. The intranet
  keeps `GithubIdentity(githubUserId ↔ sub, refreshedAt)`, upserted from the claim on every
  login; webhook resolution is a local lookup — no IdP call in the webhook path. Authentik
  remains authoritative; the table is a disposable projection.
- **Unknown author:** webhook author resolves to no `sub` → the PR event is stored
  unattributed and flagged to Ops (likely a manual PR or an unlinked reviewer), never
  dropped. Reviewer identities get the same treatment when approvals matter (§4.5).

**Rejected:** (B) self-declared GitHub handle on the intranet profile — unverified, drifts,
duplicates state; (C) Auth.js `Account` rows — structurally empty of GitHub data (see
invariant); (D) reading the mapping via the Authentik REST API at runtime — works, but
adds a service token to custody and a runtime dependency for no benefit over the claim;
keep as fallback if the claim approach hits a wall.

## Implementation notes for ENG-330 (spike gotchas)

- Source-enrolled users are created with type **external** and cannot open the Authentik
  UI; the enrollment flow must set type *internal* (or an equivalent flow policy) or the
  Flow-3 "Connect GitHub" deep link breaks for Google-enrolled teachers.
- OAuth2 providers created via API need explicit
  `grant_types: ["authorization_code", "refresh_token"]`.
- The `github_id` scope mapping only refreshes **at token issuance** — after linking
  GitHub mid-session the intranet must trigger the silent re-auth before re-checking the
  gate; do not poll or cache-bust any other way.
- The scope mapping is **spike-validated** (2026-08-10, see `spikes/authentik/README.md`):
  a linked user previews `github_id: 5514150`, an unlinked user gets no value. The working
  expression (goes into the blueprint) is:

  ```python
  from authentik.sources.oauth.models import UserOAuthSourceConnection
  conn = UserOAuthSourceConnection.objects.filter(
      user=request.user, source__slug="github").first()
  return {"github_id": int(conn.identifier)} if conn else {}
  ```

- Custom claims ride on their **scope**: Auth.js must request
  `openid email profile github_id` in the provider config or the claim never arrives.
- User-source connections are **read-only via API** (POST → 405); only login/linking flows
  create them. Ops "manual linking" therefore always means driving the user through
  Settings → Connected services, or acting on the Moodle link (Flow 2), never forging a
  source connection.

## Out of scope

- FR-7 one-off migration of existing teachers/staff (deferred; Flow 2's machinery — email
  code + Ops queue — is reusable for it).
- Nostr anywhere in the IdP (stays Moodle-native).

## Review checklist (approval = ENG-326 done)

- [ ] Flow 1: accept prompt-stage confirmation, or de-scope to `email_link` + notification?
- [ ] Flow 2: email-code verification acceptable as the primary fallback?
- [x] Flow 2: Ops queue owner = Authentik admins group, via the `groups` claim
      (decided 2026-08-10, spike-verified).
- [x] Flow 3: GitHub connection as hard gate at submission time, transported as the
      `github_id` claim (agreed + spike-verified 2026-08-10).
