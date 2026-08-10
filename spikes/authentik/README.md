# Spike ENG-325 — Authentik + `sub` estable 1:1

Spike de la Story 00 (identidad federada). Valida (1) dimensionamiento/coste de Authentik
para el volumen de teachers/staff y (2) que el `sub` OIDC es estable y 1:1 con la cuenta,
independiente del método de login (local / Google / GitHub).

## Cómo levantarlo

```bash
cp .env.example .env   # o genera secretos: PG_PASS, AUTHENTIK_SECRET_KEY, AUTHENTIK_BOOTSTRAP_*
docker compose up -d   # postgres + server + worker (2026.5.6, ya sin redis)
# UI: http://localhost:9000  — admin: akadmin / AUTHENTIK_BOOTSTRAP_PASSWORD
# API: Authorization: Bearer $AUTHENTIK_BOOTSTRAP_TOKEN
```

El bootstrap por variables de entorno (`AUTHENTIK_BOOTSTRAP_PASSWORD/TOKEN/EMAIL`) crea
`akadmin` con un token de API listo para automatizar — no hace falta el wizard del navegador.

## Hallazgo 1 — `sub` estable 1:1 ✅ (mecanismo confirmado)

El `sub` que emite un provider OIDC de Authentik se calcula **solo a partir del objeto User**
(`sub_mode: hashed_user_id`, el default; hay variantes user_uuid/username/email). El método de
autenticación (contraseña local o fuente social) no interviene en el claim.

Comprobado empíricamente vía API (`/providers/oauth2/{id}/preview_user/`):

- Mismo usuario, llamadas repetidas → mismo `sub` (`ad82f2cf…a79d`).
- Usuario distinto → `sub` distinto (1:1).
- Añadir una fuente GitHub al deployment no altera el `sub` del usuario.

Verificado además con el flujo real (authorization code, 2026-08-10): login en el navegador
como `prueba-teacher` → id_token con `sub` **idéntico** al del preview. Ojo al crear
providers por API: `grant_types` queda vacío y hay que incluir explícitamente
`["authorization_code", "refresh_token"]` (la UI lo rellena sola); si no, el authorize
falla con `invalid_request` ("Invalid grant_type for provider" en los logs).

Los logins sociales crean una *user source connection* que apunta al User existente:
con `user_matching_mode: email_link` en la fuente, un login de Google/GitHub cuyo email
coincida se vincula automáticamente a la cuenta y por tanto emite el mismo `sub`.
En 2026.x estas conexiones solo las crea el flujo de login (la API devuelve 405 al POST),
así que la verificación end-to-end requiere credenciales OAuth reales:

- [x] Login local end-to-end → `sub` del id_token real coincide con el preview.
- [x] OAuth apps reales de Google y GitHub conectadas como fuentes; matriz completa
      verificada el 2026-08-10 sobre la cuenta `ifuensan` (pk 6) — los tres métodos
      devuelven el mismo `sub` (`ed8e38e2…9ce7`) en id_tokens reales:

| Método de login | auth_method en eventos | `sub` emitido |
|---|---|---|
| Google (fuente OIDC) | source google-spike | `ed8e38e2…9ce7` |
| GitHub (fuente OAuth) | source github-spike | `ed8e38e2…9ce7` |
| Contraseña local | password | `ed8e38e2…9ce7` |

**CONFIRMADO: `sub` estable 1:1 entre los tres métodos.**

Datos operativos que salieron de la prueba real:

- Email no coincidente → enrollment de cuenta nueva con tipo **external**, que no puede
  acceder a la interfaz de usuario de Authentik ("solo usuarios internos"). Los usuarios
  enrolados por fuente hay que pasarlos a `internal` (o ajustar el enrollment flow).
  Este es el caso fallback de OQ-4: la vinculación manual se hace en
  Settings → Servicios conectados del propio usuario.
- `email_link` solo vincula automáticamente si el email verificado de la fuente coincide
  con un usuario existente.

Recomendación para la intranet: persistir el `sub` tal cual (o `user_uuid` si se prefiere
un identificador no dependiente del secreto de la instancia) y **nunca** los sub de
Google/GitHub, como ya dice la story.

## Hallazgo 2 — dimensionamiento y coste ✅

Medido en esta instancia (stack completo, idle):

| Contenedor | RAM | CPU idle |
|---|---|---|
| server | ~490 MB | <0.5 % |
| worker | ~245 MB | <0.1 % |
| postgres 16-alpine | ~145 MB | ~0 % |
| **Total** | **~880 MB** | — |

Imagen del server: 1.2 GB; BD recién poblada: <100 MB. Para decenas de teachers/staff
(logins esporádicos, no tráfico de estudiantes) sobra un **VPS 2 vCPU / 4 GB** (~4–8 €/mes
en Hetzner/OVH) o convivir en la infra existente. Authentik self-hosted es gratis para este
uso (las features enterprise no aplican); desde 2025.x ya no necesita Redis, un servicio
menos que operar.

## Hallazgo 3 (ENG-326) — claim `github_id` por scope mapping ✅

Validado el mecanismo del Flujo 3 del design doc (`docs/design/00-identity-linking.md`):
un scope mapping custom (configuración, apta para blueprint) expone el ID numérico de
GitHub de la source connection como claim del id_token. Probado con `preview_user`:
`ifuensan` (GitHub vinculado) → `github_id: 5514150`; `prueba-teacher` (sin vincular) →
sin valor. El cliente debe pedir el scope `github_id` para recibir el claim. Expresión en
el design doc.

## Objetos creados en la instancia de prueba

- Usuario local `prueba-teacher` (pk 5); usuario `ifuensan` (pk 6, enrolado vía Google,
  con GitHub y Google vinculados y contraseña local)
- Provider OIDC `spike-intranet-oidc` (sub_mode `hashed_user_id`, redirect
  `http://localhost:9009/api/auth/callback/authentik`) + app `spike-intranet`
- Fuentes OAuth `github-spike` y `google-spike` (credenciales reales de prueba,
  `email_link`) — **rotar/borrar las OAuth apps al desmontar el spike**
- Scope mapping `spike-github-id-claim` (claim `github_id`)
