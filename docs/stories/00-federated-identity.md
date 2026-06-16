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

> *IdP = Identity Provider · OIDC = OpenID Connect (capa de identidad sobre OAuth 2.0) ·
> `sub` = identificador de usuario, único por IdP.*

## User story

**As** an aspiring teacher / staff member, **I want** to log into the intranet with a single
account, **so that** I don't manage separate credentials and my work isn't interrupted if
Moodle is down.

## Model (proposed)

- IdP dedicado del que cuelgan Google y GitHub. Intranet y Moodle son clientes
  independientes del IdP vía OIDC.
- El IdP es la fuente única de identidad para quien entra por la intranet. Autenticación
  (quién eres) vive en el IdP; autorización (qué puedes hacer) vive en cada sistema por
  separado.

## Acceptance criteria

- **Métodos de login (vía IdP):** usuario/contraseña (cuenta local en el IdP), Google
  (OIDC) o GitHub (OIDC); el usuario elige. Nostr no es método de la intranet; vive solo en
  Moodle (nativo). Un profesor que use Nostr en Moodle accede a la intranet con cualquiera de
  los tres métodos del IdP, sin traer una identidad social externa.
- **Identidad única:** un solo registro (la cuenta del IdP); la intranet guarda solo la
  referencia al **`sub` del IdP** (no al `sub` de Google ni de GitHub). Los tres métodos
  conviven en la misma cuenta del IdP y resuelven al mismo `sub`.
- **Independencia de fallo:** si Moodle está caído, el IdP y la intranet siguen autenticando.
  La identidad de profes/staff no depende de Moodle.
- **Provisión en primer login:** usuario nuevo entra por la landing → IdP autentica → la
  intranet crea su perfil (instructor/staff) referenciado por el `sub` del IdP.

## Entry gates (principle: the gate decides the method)

- Entrada por la intranet (landing "Conviértete en profesor", o staff) → IdP (Google/GitHub).
- Entrada por Moodle directo (estudiante a su curso) → login nativo de Moodle
  (Google/GitHub/Nostr/user-pass). Sin tocar el IdP.

## Single friction case — existing student → teacher

Ocurre en la landing. El usuario ya tiene cuenta Moodle (creada como estudiante). Al entrar
por la intranet vía IdP, si el identificador casa (mismo Google/GitHub), se vincula. Si no,
hay un paso de vinculación de su cuenta Moodle existente a la identidad del IdP. Definir la
estrategia de vinculación y el fallback.

## Scope — students out of the IdP for now

Los estudiantes siguen en Moodle nativo. El IdP es solo para profes/staff. Ampliar a
estudiantes es una fase futura (sumar usuarios al mismo IdP, sin rehacer arquitectura).

## Authorization (NOT managed by the IdP)

- El IdP aporta **identidad y el rol amplio** (staff/instructor). La autorización fina la
  gestiona cada sistema.
- Roles de intranet (propios): instructor, revisor/Ops, marketing.
- Roles de Moodle (propios, gestionados en Moodle): Student, Teacher, Manager, admin. El
  loader asigna Teacher al publicar ([P101](../platform/P101-moodle-user-role-enrolment.md));
  Manager y admin se gestionan en Moodle. El rol admin de Moodle es ortogonal a la intranet.

## Technical notes

- IdP: **Authentik self-hosted** (proporcionado al volumen acotado de profes/staff). Solo
  Google/GitHub (OAuth estándar, de fábrica); sin desarrollo de Nostr-stage, al quedar Nostr
  fuera del IdP.
- El token OIDC del IdP es para login. Las llamadas a Web Services de Moodle (crear cursos,
  asignar roles, [P101](../platform/P101-moodle-user-role-enrolment.md)) siguen usando
  `wstoken`. Autenticación e integración API son planos separados.
- Moodle 5.1: conectar Moodle al IdP como cliente OAuth2 core (config, no plugin). 5.1 exige
  mover plugins a `/public` tras upgrade.
- Migración: mapear profes/staff existentes al `sub` del IdP (puntual, no lógica permanente).

## Recommended spike (before implementing)

Validar Authentik para el volumen profes/staff, conectar Moodle 5.1 como cliente OIDC,
confirmar `sub` estable 1:1, y la estrategia de vinculación estudiante→profesor. (Se
simplifica respecto a versiones previas: sin pieza Nostr.)

## Open questions

- **Estrategia de vinculación estudiante→profesor** cuando el identificador NO casa — definir
  el flujo y el fallback (PRD OQ-4, build-blocking, → architecture).

## Resolved

- **¿Authentik full en MVP?** — RESUELTO (PRD, decision-log Q1): se va por **federación completa
  de 3 métodos en v1** (local + Google + GitHub vía Authentik self-hosted), con 2 devs dedicados.
  No se toma el piloto GitHub-OIDC-only.
