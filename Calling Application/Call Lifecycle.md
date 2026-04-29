---
tags: [calling-app, architecture, call-flow, twilio]
---

# Call Lifecycle

> [[00 - Index|← Back to Index]]

## Outbound Call — Full Flow

```mermaid
sequenceDiagram
    participant AG as Agent Browser
    participant FE as calling_application_fe
    participant BE as twilio-voice-service
    participant TW as Twilio
    participant CU as Customer Phone

    FE->>BE: GET /api/token (JWT auth)
    BE->>BE: Check wallet balance ≥ $1
    BE-->>FE: Twilio Access Token
    FE->>TW: device.setup(token) — register browser
    TW-->>FE: Device ready

    AG->>FE: Dials a number
    FE->>TW: device.connect({ To: "+1234567890" })
    TW->>BE: POST /voice (TwiML webhook)
    BE->>BE: Create conference: conf_{callSid}
    BE-->>TW: TwiML <Conference conf_{callSid}>
    TW-->>AG: Agent joins conference

    BE->>TW: REST API: dial customer +1234567890
    TW->>BE: POST /voice/join-conference (customer TwiML)
    BE-->>TW: TwiML <Conference conf_{callSid}>
    TW->>CU: Customer phone rings
    CU-->>TW: Customer answers
    TW-->>AG: Call is live (both in conference)

    loop Every 2 seconds
        FE->>BE: GET /api/call-logs/{sid}/transcription
        BE-->>FE: Transcription chunks
        FE->>FE: Display live transcript
        FE->>FE: Detect keywords → suggest courses
    end

    loop Every 60 seconds
        BE->>BE: billingService.js
        BE->>BE: Calculate unbilled seconds
        BE->>BE: Deduct from master wallet
        BE->>BE: Update last_billed_at
    end

    Note over AG: CRM profile shown automatically
    FE->>BE: GET /api/contacts/crm-profile/{number}
    BE-->>FE: Client CRM data

    AG->>FE: Ends call
    TW->>BE: POST /webhooks/voice/status (status=completed)
    BE->>BE: SELECT FOR UPDATE on call_log
    BE->>BE: Bill remaining seconds
    BE->>BE: Mark is_billed = true
    TW->>BE: POST /webhooks/voice/recording-complete
    BE->>BE: Save recording URL to call_logs
    FE->>AG: ACW Modal appears
    AG->>FE: Log outcome, notes, create follow-up task
```

---

## Inbound Call — Full Flow

```mermaid
sequenceDiagram
    participant CU as Customer Phone
    participant TW as Twilio
    participant BE as twilio-voice-service
    participant FE as calling_application_fe
    participant AG as Agent Browser

    CU->>TW: Calls Twilio number
    TW->>BE: POST /webhooks/voice/inbound
    BE->>BE: Lookup assigned agent for this number
    BE-->>TW: TwiML: dial agents conference
    TW->>FE: Twilio JS SDK: incoming call event
    FE->>AG: Incoming call modal (Accept / Reject)

    alt Agent accepts
        AG->>FE: Accept
        FE->>TW: call.accept()
        TW-->>AG: Agent in conference
        TW-->>CU: Customer in conference
        Note over AG,CU: Call is live — same flow as outbound
    else Agent does not answer
        TW->>BE: POST /webhooks/voice/agent-no-answer
        BE-->>TW: TwiML: redirect to voicemail
        TW->>BE: POST /voice/voicemail (TwiML)
        BE-->>TW: TwiML: record voicemail
        TW->>BE: POST /webhooks/voice/voicemail-complete
        BE->>BE: Save voicemail recording URL
    end
```

---

## Supervisor Monitoring Flow

```mermaid
sequenceDiagram
    participant SV as Supervisor Browser
    participant FE as calling_application_fe
    participant BE as twilio-voice-service
    participant TW as Twilio
    participant CF as Live Conference

    loop Every 5 seconds
        FE->>BE: GET /api/supervisor/active-calls
        BE-->>FE: List of live conferences + agent info
    end

    SV->>FE: Click "Listen" on a call
    FE->>BE: POST /api/supervisor/listen { callSid }
    BE->>TW: REST: dial supervisor into conference (muted)
    TW-->>CF: Supervisor joined (cannot be heard)

    SV->>FE: Click "Whisper"
    FE->>BE: POST /api/supervisor/whisper { callSid }
    BE->>TW: REST: unmute supervisor to agent only
    Note over CF: Supervisor → Agent only\nCustomer cannot hear

    SV->>FE: Click "Barge"
    FE->>BE: POST /api/supervisor/barge { callSid }
    BE->>TW: REST: unmute supervisor to everyone
    Note over CF: Supervisor → Agent + Customer
```

---

## Conference Participant Map

```
Twilio Conference: conf_{callSid}
├── Leg 1: Agent (browser via Twilio JS SDK)
│   └── Can be: on hold (muted from conference)
├── Leg 2: Customer (PSTN phone)
│   └── Hears: agent + supervisor if barged
└── Leg 3: Supervisor (optional)
    ├── Listen mode: muted — hears everyone
    ├── Whisper mode: unmuted to agent only
    └── Barge mode: unmuted to everyone
```

---

## Billing Calculation

```
Rate example: $0.04/minute outbound

During call (every 60s):
  unbilled_seconds = NOW() - last_billed_at
  amount = CEIL((unbilled_seconds / 60) * 0.04 * 100) / 100
  → Deduct from wallet_transactions

On call end:
  final_seconds = NOW() - last_billed_at
  if not is_billed:
    Final deduction
    is_billed = true
```

**Lock mechanism:** `SELECT FOR UPDATE` on `call_logs` prevents double-billing if webhook fires twice.

---

## Real-Time Features Summary

| Feature | Mechanism | Interval |
|---------|-----------|---------|
| Live transcription | Frontend polls `/call-logs/{sid}/transcription` | Every 2s |
| Wallet balance | Frontend polls `/wallet/balance` | Every 30s |
| Active calls (supervisor) | Frontend polls `/api/supervisor/active-calls` | Every 5s |
| Billing deduction | Backend setInterval in billingService.js | Every 60s |
| Transcription delivery | Twilio POSTs to `/webhooks/voice/transcription` | Real-time |

No WebSockets anywhere — all polling or Twilio webhook callbacks.

---

## Connects To

- [[twilio-voice-service]] — the backend powering this
- [[calling_application_fe]] — the frontend UI
- [[Twilio Integration]] — Twilio APIs used
- [[Database Schema]] — call_logs, conferences, wallet_transactions
