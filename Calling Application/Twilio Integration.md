---
tags: [calling-app, integration, twilio, calling]
---

# Twilio Integration

> [[00 - Index|← Back to Index]]

## What Twilio Does In This System

Twilio provides the entire telephony layer:
- **Browser calling** (WebRTC) via Twilio JS SDK
- **PSTN dialing** (calling real phone numbers)
- **Conference management** (multi-party calls)
- **Call recording** (automatic, downloadable)
- **Live transcription** (Google engine)
- **Voicemail** (record + store)

---

## Twilio Account Resources Needed

| Resource | Env Var | Purpose |
|----------|---------|---------|
| Account SID | `TWILIO_ACCOUNT_SID` | Identity for REST API |
| Auth Token | `TWILIO_AUTH_TOKEN` | REST API authentication |
| API Key | `TWILIO_API_KEY` | Access Token generation |
| API Secret | `TWILIO_API_SECRET` | Access Token signing |
| TwiML App SID | `TWILIO_TWIML_APP_SID` | Routes browser calls to `/voice` |
| Phone Number | `TWILIO_PHONE_NUMBER` | Default outbound caller ID |

---

## How Browser Calling Works (TwiML + SDK)

```mermaid
flowchart TD
    A["1. Agent loads app"] --> B["GET /api/token\nBackend creates AccessToken\nwith VoiceGrant"]
    B --> C["2. Twilio JS SDK\ndevice.setup(token)\nBrowser registers as endpoint"]
    C --> D{"3. Call direction"}
    D -->|"Outbound"| E["device.connect({ To: phone })\nTwilio routes to TwiML App SID"]
    D -->|"Inbound"| F["Twilio calls TwiML App URL\nSDK fires incoming event"]
    E --> G["4. Twilio hits POST /voice\n(TwiML webhook)"]
    F --> G
    G --> H["5. Backend returns TwiML\n<Conference name=conf_XYZ>"]
    H --> I["6. Twilio creates conference\nAgent browser joins"]
    I --> J["7. Backend dials customer\nvia REST API"]
    J --> K["8. Customer phone rings\nJoins same conference"]
    K --> L["Live call in Twilio Conference"]
```

---

## TwiML Responses

TwiML (Twilio Markup Language) — XML that instructs Twilio what to do:

**Outbound call (agent dials customer):**
```xml
<Response>
  <Dial>
    <Conference name="conf_{callSid}"
                startConferenceOnEnter="true"
                endConferenceOnExit="true"
                record="record-from-start"
                transcribe="true">
    </Conference>
  </Dial>
</Response>
```

**Voicemail:**
```xml
<Response>
  <Say>Please leave a message after the beep.</Say>
  <Record maxLength="120" action="/webhooks/voice/voicemail-complete"/>
</Response>
```

**Hold music:**
```xml
<Response>
  <Play loop="0">https://your-server.com/hold-music</Play>
</Response>
```

---

## Conference API (Supervisor Controls)

Once a conference exists, [[twilio-voice-service]] uses the Twilio REST API to control participants:

| Action | Twilio REST Call |
|--------|-----------------|
| Put on hold | `PATCH /conferences/{conf}/participants/{callSid}` `{ muted: true }` + play hold music |
| Unhold | `PATCH /conferences/{conf}/participants/{callSid}` `{ muted: false }` + stop hold music |
| Supervisor listen | Dial supervisor into conference with `muted: true` |
| Supervisor whisper | `PATCH` to unmute supervisor, update customer leg to hear only agent |
| Supervisor barge | `PATCH` to unmute supervisor for all participants |
| End call | `DELETE /conferences/{conf}/participants/{callSid}` |

---

## Webhooks

Twilio sends HTTP POST callbacks to `BASE_URL` for all events:

```mermaid
sequenceDiagram
    participant TW as Twilio
    participant BE as twilio-voice-service

    TW->>BE: POST /webhooks/voice/inbound\n{ CallSid, From, To, CallStatus }
    TW->>BE: POST /webhooks/voice/status\n{ CallSid, CallStatus: "in-progress"|"completed" }
    TW->>BE: POST /webhooks/voice/transcription\n{ TranscriptionText, CallSid, Track }
    TW->>BE: POST /webhooks/voice/recording-complete\n{ RecordingSid, RecordingUrl, RecordingDuration }
    TW->>BE: POST /webhooks/voice/agent-no-answer
    TW->>BE: POST /webhooks/voice/voicemail-complete\n{ RecordingUrl }
    TW->>BE: POST /webhooks/conference/status\n{ ConferenceSid, FriendlyName, StatusCallbackEvent }
```

**Webhook Signature Validation:** All webhook handlers validate `X-Twilio-Signature` header to ensure the request came from Twilio and not a third party.

---

## Transcription

Twilio handles transcription via **Google Speech engine**:
- Each speech segment → `POST /webhooks/voice/transcription`
- Stored in `call_transcriptions` table with: `track` (inbound/outbound), `transcript`, `confidence`, `sequence_number`
- Frontend polls `GET /api/call-logs/{sid}/transcription` every 2 seconds
- Keywords detected client-side → trigger course suggestion AI call

---

## Recording

- Recordings start automatically (`record="record-from-start"` in TwiML)
- When complete → Twilio POSTs to `/webhooks/voice/recording-complete`
- Recording URL stored in `call_logs.recording_url`
- Download: `GET /api/call-logs/{sid}/recording` → backend proxies audio from Twilio (authenticated URL)

---

## Inbound Call Routing

```mermaid
flowchart TD
    A["Customer calls Twilio number"] --> B["Twilio POSTs to /webhooks/voice/inbound"]
    B --> C["Lookup: which agent owns this number?"]
    C --> D{Agent found?}
    D -->|"Yes"| E["Return TwiML: dial agent conference"]
    D -->|"No"| F["Return TwiML: sorry, try again"]
    E --> G{Agent response}
    G -->|"Answers"| H["Call connected — both in conference"]
    G -->|"Clicks Transfer to AI"| I["POST /voice/transfer-to-ai\nRedirect customer → Lisa"]
    G -->|"No answer timeout"| J["handleStatus webhook\nRedirect customer → /voice/ai-answer"]
    I --> K["Lisa handles call\nLead + task created on end"]
    J --> K
```

---

## AI Call Routing (Lisa)

When a call is routed to Lisa, Twilio uses the existing call leg — no new call is created:

```
calls(customerCallSid).update({ url: BASE_URL + '/voice/ai-answer' })
```

This redirects the customer's in-progress call to a new TwiML document. The AI flow then runs as a Gather → Respond loop on that same call leg.

**Conference for monitoring:** A conference row `ai_{customerCallSid}` is created in the DB and as a real Twilio conference. Agents/supervisors can join it muted to hear Lisa's replies via `announceUrl`.

---

## Phone Number Assignment

Multiple agents can share one Twilio phone number:
- `phone_numbers` table — stores Twilio numbers
- `user_phone_assignments` table — maps agents to numbers (many-to-one)
- On inbound call: lookup which agent to route to based on the called number

---

## Connects To

- [[twilio-voice-service]] — implements all Twilio logic
- [[calling_application_fe]] — uses Twilio JS SDK in browser
- [[Call Lifecycle]] — full call flow using Twilio
- [[Database Schema]] — call_logs, conferences, call_transcriptions tables
