# SharingBridge — Community meal coordination platform

> Affordable meals with dignity — for anyone who needs food, and for the people who arrange or pay for it

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**Live web:** [https://sharingbridge.org](https://sharingbridge.org) · **Docs repo:** this repository · **Progress:** [STATUS.md](development/STATUS.md)

## What it is

SharingBridge helps people **arrange meals** — for themselves, family, seniors, neighbours, or anyone they meet who needs food. Initiators use a **Flutter** mobile app; ops/neighbourhood views use a **React** web dashboard. The platform tracks **intent and handover**; **payments stay with vendors** (Swiggy, Zomato, eco kitchens). SharingBridge is never the merchant of record.

**Shipped today (MVP):** Google sign-in, Help a seeker (direct order), eco-kitchen routes (pledge / I pay), Actions board, Connection + FCM, handover map + reverse geocode, AI instruction packs (Groq/Gemini), neighbourhood dashboard. Details: [STATUS.md](development/STATUS.md).

## Architecture (simple)

```text
Flutter mobile ──┐
                 ├──► integration-service (Experience API / BFF)
Vite/React web ──┘           │
                               ├──► user-service → Postgres (auth, presets)
                               ├──► ai-orchestration → Groq / Gemini
                               ├──► photo-service → Cloudinary
                               ├──► notification-service → FCM
                               └──► Postgres / PostGIS (order intents, marketplace)
                                         ↓
                              Vendor deep links (payment off-platform)
```

| Piece | Tech | Why (short) |
|-------|------|-------------|
| Mobile | Flutter | One app for Android field flows |
| Web | Vite + React | Static dashboard on Render |
| Experience API | Node 20 HTTP | Small BFF; fast free-tier deploys |
| Auth | user-service + Google JWT | Identity separate from journeys |
| AI | FastAPI + Groq/Gemini | Live enrichment required; fail closed if LLM down; prompts include content-safety rules |
| Photos | Cloudinary | Managed uploads without our own object store |
| Data | Supabase Postgres + PostGIS | Geo neighbourhood feeds |
| Host | Render + `sharingbridge.org` | Low-cost Git-linked hosting |

**Full stack + rationale:** [Technical Architecture § As-built](design/SharingBridge_Technical_Architecture.md#as-built-architecture-july-2026).

## Repositories (shipped)

| Repo | Role |
|------|------|
| `sharingbridge` (this) | Docs and coordination |
| `sharingbridge-mobile-app` | Flutter initiator app |
| `sharingbridge-web-app` | Vite + React dashboard |
| `sharingbridge-integration-service` | Experience API / BFF |
| `sharingbridge-user-service` | Auth + vendor presets |
| `sharingbridge-ai-orchestration` | LLM pipelines |
| `sharingbridge-photo-service` | Photo upload |
| `sharingbridge-notification-service` | FCM on kitchen commit |

**Not started:** `api-gateway`, `order-service`, `infra`. **Archived:** `location-safety`.

## Documentation map

When docs disagree, **higher row wins**.

| Layer | Doc | Owns |
|-------|-----|------|
| 1 | [BRD](requirements/SharingBridge_Business_Requirement.md) | Operating constraints |
| 2 | [PRODUCT_MODEL.md](development/PRODUCT_MODEL.md) | Vocabulary, actors, routes |
| 3 | [Eco_Kitchen_Initiation_Flow.md](design/Eco_Kitchen_Initiation_Flow.md) | Three initiation routes |
| 4 | [Technical Architecture § As-built](design/SharingBridge_Technical_Architecture.md#as-built-architecture-july-2026) | Stack & services **today** |
| 5 | [STATUS.md](development/STATUS.md) | Shipped vs plan |
| 6 | [configuration/](configuration/) | Run, deploy, env, SQL |
| 7 | [ENGINEERING_PLAN.md](development/ENGINEERING_PLAN.md) | Long-term / scale plan (aspirational) |
| 8 | [Future_Extensions.md](design/Future_Extensions.md) | Recurring orders, delivery proof, BOM vision |

### Quick links

| I want to… | Open |
|------------|------|
| See what is shipped | [STATUS.md](development/STATUS.md) |
| Deploy / run | [configuration/README.md](configuration/README.md) → [e2e-deployment-sequence.md](configuration/e2e-deployment-sequence.md) |
| SQL order | [database-setup-sequence.md](configuration/database-setup-sequence.md) |
| Manual test | [MANUAL_TESTING_GUIDE.md](testing/MANUAL_TESTING_GUIDE.md) |
| Product terms | [PRODUCT_MODEL.md](development/PRODUCT_MODEL.md) |
| Agent next tasks | [AGENT_SESSION.md](development/AGENT_SESSION.md) |
| Handover map | [field-handoff.md](configuration/field-handoff.md) → [Location ADR](design/Location_Services_Vendor_Abstraction.md) → [Map picker](design/Handover_Location_Map_Picker.md) |

**Language:** prefer **initiator / payer / beneficiary** in prose — not donation/alms framing. Legacy API names (`donor_*`) remain until renamed. See [PRODUCT_MODEL.md](development/PRODUCT_MODEL.md) § Documentation verbiage.

## Security (as-built)

- Google Sign-In → JWT (HS256) shared across services  
- CORS allowlist on user-service + integration-service (`WEB_CORS_ORIGINS`, comma-separated)  
- No platform payment ledger; minimize payment-related storage (BRD)  
- Reference photos via Cloudinary; **face-match / delivery proof capture not shipped**

## Getting started

1. [STATUS.md](development/STATUS.md) — what works today  
2. [e2e-deployment-sequence.md](configuration/e2e-deployment-sequence.md) — Phases 0–5  
3. [database-setup-sequence.md](configuration/database-setup-sequence.md) — SQL order  
4. [MANUAL_TESTING_GUIDE.md](testing/MANUAL_TESTING_GUIDE.md) — verify  

## Contributing

Technical and non-technical contributors welcome — [CALL_FOR_CONTRIBUTORS.md](development/CALL_FOR_CONTRIBUTORS.md). AI-assisted sessions are coordinated in [AGENT_SESSION.md](development/AGENT_SESSION.md).

## License

MIT — see [LICENSE](LICENSE).
