# AI — as-built wiring

**Purpose:** How AI is **wired today** (not a future plan). Phased roadmap and provider split: [AI_PLAN.md](./AI_PLAN.md). Progress snapshot: [STATUS.md](./STATUS.md) § AI.

**Last updated:** July 2026

---

## Short answer

**Yes — AI is wired end-to-end** when orchestration is running and integration flags are enabled (defaults in `sharingbridge-integration-service/env.example`):

- `AI_SUGGEST_VENDORS_ENABLED=true`
- `AI_INSTRUCTION_PACK_ENABLED=true`
- `AI_ORCHESTRATION_BASE_URL` → `sharingbridge-ai-orchestration`

| Mode | Behavior |
|------|----------|
| **`AI_LLM_MODE=live`** + Groq (+ Gemini for vision) | Live enrichment. User text is reject-gated for inappropriate content (**400**). System prompts include content-safety rules. |
| **LLM unreachable / mode not live** | **503** — fail closed; raw user text is **not** echoed downstream |
| Flags off or orchestration unreachable | Integration returns **503** |

Clients **never** call model APIs directly. Flow: **mobile/web → integration-service → ai-orchestration → Groq/Gemini**.

Hardcoded sample vendors (A2B, …) live only under **unit-test fixtures**.

Reverse geocode (`GET /v1/geocode/reverse`, Nominatim) lives on **integration-service**, not ai-orchestration.

---

## Architecture

```text
sharingbridge-mobile-app / sharingbridge-web-app
              │
              ▼
   sharingbridge-integration-service  (Experience API)
              │  AI_ORCHESTRATION_BASE_URL
              ▼
   sharingbridge-ai-orchestration  (FastAPI)
              ├── Groq  (text: suggest-vendors, instruction compose)
              └── Gemini (vision: reference photo → descriptions)

   sharingbridge-photo-service  (reference upload — separate HTTP from mobile)
   Nominatim  ← called by integration-service /v1/geocode/reverse (not AI)
```

**Not used:** LangChain in shipped code (direct HTTP/SDK). **Deferred:** `sharingbridge-location-safety` (archived).

Public integration endpoints:

- `POST /v1/donor-setup/suggest-vendors`
- `POST /v1/donor-seeker/instruction-pack`

Orchestration exposes matching internal routes; see `sharingbridge-ai-orchestration/app/main.py`.

---

## Environment (minimum)

**integration-service:**

```env
AI_ORCHESTRATION_BASE_URL=http://localhost:8091
AI_ORCHESTRATION_INTERNAL_API_KEY=<shared secret>
AI_SUGGEST_VENDORS_ENABLED=true
AI_INSTRUCTION_PACK_ENABLED=true
```

**ai-orchestration:**

```env
AI_LLM_MODE=live          # required; non-live fails closed (no raw user-text echo)
GROQ_API_KEY=
GEMINI_API_KEY=
PHOTO_SERVICE_BASE_URL=http://localhost:8092
NOMINATIM_USER_AGENT=SharingBridge-Local/1.0 (you@example.com)
```

Step-by-step keys and Render: [ai-setup-handhold.md](../configuration/ai-setup-handhold.md) · [ai-orchestration-local.md](../configuration/ai-orchestration-local.md).

---

## Still open (not “unwired”)

| Item | Status |
|------|--------|
| Delivery photo + face match | Not built (photo-service upload only) |
| LangChain / LangServe | Not used; direct SDK |
| LangSmith tracing | Optional, not required for MVP |

Future phases: [AI_PLAN.md](./AI_PLAN.md).

---

## Related docs

| Doc | Use |
|-----|-----|
| [AI_PLAN.md](./AI_PLAN.md) | Provider split, future phases |
| [Donor_Setup_AI_Search_Sequence.md](../design/Donor_Setup_AI_Search_Sequence.md) | suggest-vendors sequence |
| [field-handoff.md](../configuration/field-handoff.md) | Help a seeker flow |
| [MANUAL_TESTING_GUIDE.md](../testing/MANUAL_TESTING_GUIDE.md) | §2a orchestration smoke |
