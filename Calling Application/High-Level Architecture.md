---
tags: [calling-app, architecture, overview]
---

# High-Level Architecture

> [[00 - Index|← Back to Index]]

## System Architecture Diagram

```mermaid
graph TB
    subgraph Frontends
        FE1["🖥️ calling_application_fe\nReact + Vite\nPort 5173"]
        FE2["💰 finance-service\nReact + Vite\nPort 3001"]
    end

    subgraph Backends
        BE1["📞 twilio-voice-service\nExpress\nPort 3000"]
        BE2["🏢 crm-service-next\nNext.js\nPort 4000"]
        BE3["🔧 crm-backend\nExpress\nPort 4000"]
    end

    subgraph External
        TW["📱 Twilio API\nCalls, SMS, Webhooks"]
        AI["🤖 Anthropic\nClaude AI"]
        GM["📧 Gmail\nSMTP + IMAP"]
    end

    DB[("🗄️ PostgreSQL\ntwilio_app\nShared DB")]

    FE1 -->|"VITE_API_BASE_URL\nAll calling ops"| BE1
    FE1 -.->|"VITE_CRM_URL\nnav link"| BE2
    FE1 -.->|"VITE_FINANCE_URL\nnav link"| FE2

    FE2 -->|"/api/* proxy\nAll finance ops"| BE3

    BE1 --> DB
    BE2 --> DB
    BE3 --> DB

    BE1 <-->|"REST API +\nWebhooks"| TW
    BE1 -->|"Claude Sonnet 4.6"| AI
    BE2 -->|"Claude Opus/Sonnet/Haiku"| AI
    BE2 -->|"Send email"| GM
    BE2 -->|"Sync inbox"| GM
```

---

## Services Summary

```mermaid
graph LR
    A["[[calling_application_fe]]"] -->|calls| B["[[twilio-voice-service]]"]
    C["[[finance-service]]"] -->|calls| D["[[crm-backend]]"]
    E["[[crm-service-next]]"] -->|internal DB| F[(PostgreSQL)]
    B -->|shared DB| F
    D -->|shared DB| F
```

---

## Shared Database

All 5 services share **one PostgreSQL database** called `twilio_app`.

See [[Database Schema]] for full table breakdown.

---

## Key Design Decisions

| Decision | Why |
|----------|-----|
| Shared DB | Single source of truth — no sync needed between services |
| Conference-based calling | Allows supervisor listen/whisper/barge without disrupting call |
| Incremental billing every 60s | Prevents loss of revenue if call drops |
| JWT stateless auth | No session store needed; each service validates independently |
| Separate finance frontend | Finance is admin-only, isolated from agent workflow |

---

## Port Map (Local Dev)

| Service | Port |
|---------|------|
| [[twilio-voice-service]] | 3000 |
| [[calling_application_fe]] | 5173 |
| [[crm-service-next]] | 4000 |
| [[crm-backend]] | 4000 ⚠️ conflict with crm-service-next |
| [[finance-service]] | 3001 |
| PostgreSQL | 5432 |
