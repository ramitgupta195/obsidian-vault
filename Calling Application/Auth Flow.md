---
tags: [calling-app, architecture, auth, jwt]
---

# Auth Flow

> [[00 - Index|← Back to Index]]

## Overview

All services use **JWT (JSON Web Tokens)** with Bearer token auth. There is **no single SSO** — each service has its own login and token. All services share the same `JWT_SECRET` and the same `users` table in PostgreSQL.

---

## Three Auth Systems (Today)

| Service | localStorage Key | JWT Expiry | Admin Check |
|---------|-----------------|-----------|------------|
| [[calling_application_fe]] → [[twilio-voice-service]] | `app_jwt` | 8 hours | `role === "admin"` |
| [[crm-service-next]] | `crm_token` | 12 hours | `role === "admin"` |
| [[finance-service]] → [[crm-backend]] | `finance_token` | 12 hours | `role === "admin"` only |

---

## Login Flow (Same for All Services)

```mermaid
sequenceDiagram
    participant U as User
    participant FE as Frontend
    participant BE as Backend

    U->>FE: Enter email + password
    FE->>BE: POST /api/auth/login { email, password }
    BE->>BE: Query users table by email
    BE->>BE: bcrypt.compare(password, password_hash)
    Note over BE: Dummy hash used for non-existent users\n(prevents timing attacks)
    BE-->>FE: { token, user: { id, email, role, name } }
    FE->>FE: Store token in localStorage
    FE->>FE: Decode JWT payload (no sig verify)
    FE->>FE: Check exp claim — if expired, clear + redirect
    U-->>FE: Redirected to dashboard
```

---

## Every API Request

```mermaid
sequenceDiagram
    participant FE as Frontend
    participant BE as Backend

    FE->>BE: GET /api/anything\nAuthorization: Bearer <jwt>
    BE->>BE: Extract token from header
    BE->>BE: jwt.verify(token, JWT_SECRET)
    BE->>BE: Attach req.user = { id, email, role }
    BE-->>FE: Response data

    alt 401 Unauthorized
        BE-->>FE: 401 error
        FE->>FE: Clear localStorage
        FE->>FE: Redirect to /login
    end
```

---

## Role-Based Access

### twilio-voice-service Roles

```mermaid
graph TD
    SA["super_admin\n= admin (equivalent)"] 
    AD["admin"]
    SM["sales_manager"]
    SP["salesperson"]

    SA -->|"can do everything"| ALL["All routes"]
    AD -->|"can do everything"| ALL
    SM -->|"team calls, reports, AI forecast"| MANAGER["Manager routes"]
    SP -->|"own calls, own contacts only"| AGENT["Agent routes"]
```

| Role | Scoped to |
|------|-----------|
| `salesperson` | Own calls + own contacts |
| `sales_manager` | Team visibility + reports |
| `admin` / `super_admin` | Everything + users, billing, phone numbers |

### crm-service-next Roles
- `admin` — full access
- Other roles — regular CRM access

### crm-backend / finance-service
- `admin` only — finance-service rejects non-admin at login

---

## Twilio Access Token (separate from auth JWT)

Used to authenticate the browser with Twilio directly (WebRTC).

```mermaid
sequenceDiagram
    participant FE as calling_application_fe
    participant BE as twilio-voice-service
    participant TW as Twilio

    FE->>BE: GET /api/token\nAuthorization: Bearer <app_jwt>
    BE->>BE: Check wallet balance ≥ $1.00
    BE->>BE: Create Twilio AccessToken\n(API Key + TwiML App SID + agent identity)
    BE-->>FE: { token: "<twilio_token>", balance, low_balance }
    FE->>TW: device.setup(twilio_token)
    Note over FE,TW: Browser now registered as Twilio endpoint
    TW-->>FE: tokenWillExpire event (before expiry)
    FE->>BE: GET /api/token (refresh)
```

---

## Security Notes

| Aspect | Detail |
|--------|--------|
| Token storage | localStorage (not httpOnly cookie — XSS risk, but standard for SPAs) |
| Password hashing | bcryptjs, 12 salt rounds |
| Timing attack prevention | Dummy hash compared even for non-existent users |
| JWT algorithm | HS256 (shared secret) |
| Rate limiting | Login endpoint: 10 attempts / 15 minutes |
| SQL injection | Parameterized queries throughout |

---

## Unification Note

In a merged app, there would be **one login, one token, one localStorage key**. The three separate auth systems exist because the services were built independently.

See [[Merge Strategy]] for how to consolidate.
