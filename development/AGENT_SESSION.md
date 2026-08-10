# SharingBridge — agent session

> **AI session doc.** Update **Next tasks** and **Recently shipped** when work lands. **Do not** duplicate progress tables here — use [STATUS.md](./STATUS.md).

## Goal

MVP **initiator vendor presets → Help a seeker / eco kitchen initiation → vendor redirect → delivery confirmation**, plus **eco kitchen** routes (Actions, Connection, FCM). Legacy code paths still use `donor_*` module names. **`sharingbridge-integration-service`** is the Experience API (BFF). SharingBridge is never the system of record for money.

## Locked approach

- Payments: provider/vendor-hosted only; no platform ledger.
- Mobile: Flutter; Experience API: Node 20 HTTP today (Spring Boot later for integration).
- Auth / System: **user-service is ASP.NET Core 8 (C#)** on Render Docker (Endpoints / Services / Repositories layout).
- Notifications: Node today → **Spring Boot** next.
- Preferences: user-service Postgres authority; client cache non-authoritative.
- Labels: Experience API = integration-service; Process = ai-orchestration, photo-service, notification-service; System = user-service + Postgres.
- **DB access:** env-driven pool/retry (`DB_POOL_*`, `DB_RETRY_*`) shipped on user-service; apply the same names to other Postgres clients in follow-up work. Prefer Supabase **session** pooler for long-lived APIs.

Full progress vs plan: **[STATUS.md](./STATUS.md)**.

---

## Documentation map

| If you need… | Read |
|--------------|------|
| **Progress vs plan (update this when shipping)** | [STATUS.md](./STATUS.md) |
| **Engineering plan (long-term)** | [ENGINEERING_PLAN.md](./ENGINEERING_PLAN.md) |
| **Product vocabulary** | [PRODUCT_MODEL.md](./PRODUCT_MODEL.md) |
| **Run / deploy order** | [configuration/e2e-deployment-sequence.md](../configuration/e2e-deployment-sequence.md) |
| **SQL order** | [configuration/database-setup-sequence.md](../configuration/database-setup-sequence.md) |
| **Manual tests** | [testing/MANUAL_TESTING_GUIDE.md](../testing/MANUAL_TESTING_GUIDE.md) |
| **Eco kitchen flows** | [design/Eco_Kitchen_Initiation_Flow.md](../design/Eco_Kitchen_Initiation_Flow.md) |
| **Handover location** | [README.md § Quick links](../README.md#quick-links) — field-handoff → vendor ADR → map picker → mobile-client |
| **AI as-built** | [AI_AS_BUILT.md](./AI_AS_BUILT.md) |
| **AI future phases** | [AI_PLAN.md](./AI_PLAN.md) |
| **Full doc index** | [README.md](../README.md) |

Prefer `configuration/*` and `MANUAL_TESTING_GUIDE.md` over per-repo READMEs for runbooks.

---

## Quick runbook

```text
integration-service   npm test && npm start                          → :8080
user-service          dotnet test && dotnet run --project src/SharingBridge.UserService → :8081
ai-orchestration      pytest -q && uvicorn…                          → :8091
photo-service         (venv) pytest && uvicorn                       → :8092
notification-service  npm test && npm start                          → :8093
web-app               npm test && npm run dev                         → :5173
mobile                flutter test && flutter run (see mobile-client.md)
```

Google sign-in and emulator URLs: [configuration/mobile-client.md](../configuration/mobile-client.md). Dev JWT: `dotnet run --project tools/MintDevJwt -- demo-user initiator` in user-service (same `AUTH_TOKEN_SECRET`).

---

## GitHub org (`sharingbridge`)

| Role | Slug |
|------|------|
| Coordination / docs | `sharingbridge` |
| Core APIs | `sharingbridge-user-service`, `sharingbridge-integration-service` |
| Clients | `sharingbridge-mobile-app`, `sharingbridge-web-app` |
| Process services | `sharingbridge-ai-orchestration`, `sharingbridge-photo-service`, `sharingbridge-notification-service` |
| Not MVP | `sharingbridge-api-gateway`, `sharingbridge-order-service`, `sharingbridge-location-safety` (archived) |

---

## Next recommended tasks

1. **Roll out `DB_POOL_*` / `DB_RETRY_*`** to integration / photo / notification — [environment-variables.md § Database client pool & retry](../configuration/environment-variables.md#database-client-pool--retry-standard).
2. **notification-service → Spring Boot** — same webhook contract; Docker on Render; adopt `DB_*` knobs.
3. **UX redesign** — clearer flows / less scrolling (web + mobile); terminology cleanup in UI.
4. **Transactional email** — Resend/SendGrid in notification-service (after Spring rewrite preferred).
5. **Order ops + delivery proof** — [Future_Extensions.md](../design/Future_Extensions.md) Phase B.
6. **Marketplace F** — beneficiary profile (see [ENGINEERING_PLAN.md](./ENGINEERING_PLAN.md) § F).
7. **APK rebuild** — `WEB_DASHBOARD_URL=https://sharingbridge.org`.

After shipping, update [STATUS.md](./STATUS.md) workstream table.

---

## Post-ship checklist

1. CI green on `main` for every repo you changed.
2. Short smoke from [MANUAL_TESTING_GUIDE.md](../testing/MANUAL_TESTING_GUIDE.md).
3. Update [STATUS.md](./STATUS.md) if a workstream status changed.
4. Add a line under **Recently shipped** below.

---

## Recently shipped (newest last)

- `feat` (web): **Updates** banner — connection-ready rows on sign-in/Refresh; Open Connection → Actions.
- `docs`: Progressive SQL setup; notification deploy path; manual guide §4d-b.
- `feat`: Eco kitchen phases 1–6; Connection API; FCM webhook; mobile **Eco kitchen · I pay**.
- `feat`: Marketplace M1–M4 in code; Actions tab; seeker demands; kitchen commit.
- `feat`: AI orchestration wired (deterministic + `AI_LLM_MODE=live` Groq/Gemini).
- `feat`: Postgres-only persistence; user-service preferences authority.
- `feat`: Google Sign-In; coordinator dashboard; Initiations / Actions / Map.
- `feat` (mobile): handover map picker + `GET /v1/geocode/reverse`; `HANDOVER_MAP_ENABLED` from `local.properties`.
- `feat` (web): Initiations / Actions mobile layout fixes.
- `docs`: location vendor ADR; unified reading sequence steps 10–13.
- `fix` (mobile): Android build toolchain — AGP 8.11.1, Gradle 8.14, Kotlin 2.3.20, `compilerOptions` DSL.
- `ops`: custom domain `sharingbridge.org` + `www` live (GoDaddy DNS → Render); Google origins + comma-separated `WEB_CORS_ORIGINS` on both backends.
- `docs`: Future_Extensions Phase C (recurring orders) + Phase D summary; PRODUCT_MODEL § Producer supply & recipe BOM; ENGINEERING_PLAN phases J–K.
- `feat` (web): Help dialog + GitHub README on sign-in; simplified sign-in copy.
- `docs`: README + As-built architecture rewritten (stack + why); trimmed outdated README claims; STATUS remains source of truth for shipped.
- `feat` (user-service): C# / ASP.NET Core 8 on Render Docker; Supabase session pooler; env-driven `DB_POOL_*` / `DB_RETRY_*` (standard for other Postgres clients next).
- `docs`: DB client pool/retry standard; session vs transaction pooler guidance for Npgsql vs Node.
- `chore` (user-service): remove dead Node `legacy-node/`; Endpoints/Services/Repositories layout; `tools/MintDevJwt`.

Older history: git log on `sharingbridge` and service repos.
