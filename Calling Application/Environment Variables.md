---
tags: [calling-app, reference, environment, config]
---

# Environment Variables

> [[00 - Index|← Back to Index]]

## twilio-voice-service

```env
# Server
PORT=3000
DATABASE_URL=postgresql://user:pass@host:5432/twilio_app
JWT_SECRET=your_jwt_secret_here

# CORS — comma-separated allowed origins
FRONTEND_URL=http://localhost:5173,https://your-fe.vercel.app

# Twilio — get from console.twilio.com
TWILIO_ACCOUNT_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_AUTH_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_API_KEY=SKxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_API_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_TWIML_APP_SID=APxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_PHONE_NUMBER=+1xxxxxxxxxx

# Public webhook URL — MUST be publicly reachable by Twilio
# In dev: use ngrok → https://xxx.ngrok-free.app
# In prod: your Render/server URL
BASE_URL=https://your-public-url.com

# AI (optional but needed for CRM AI features)
ANTHROPIC_API_KEY=sk-ant-api03-...
```

---

## calling_application_fe

```env
# Backend URL — points to twilio-voice-service
VITE_API_BASE_URL=https://your-twilio-backend.render.com

# Navigation links (browser redirects, not API calls)
VITE_CRM_URL=https://crm-service-next.vercel.app
VITE_FINANCE_URL=https://finance-service-three.vercel.app
```

**Dev proxy:** In development, Vite proxies `/api/*` to `VITE_API_TARGET` (defaults to `http://localhost:3000`) so you can omit `VITE_API_BASE_URL` locally.

---

## crm-service-next

```env
# Server
PORT=4000
DATABASE_URL=postgresql://user:pass@host:5432/twilio_app
JWT_SECRET=your_jwt_secret_here

# Public URL of calling app (for navigation link)
NEXT_PUBLIC_CALLING_APP_URL=https://your-calling-app.vercel.app

# Email — Gmail app passwords (not your real Gmail password)
# Generate at: myaccount.google.com/apppasswords
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_SECURE=false
SMTP_USER=your@gmail.com
SMTP_PASS=xxxx xxxx xxxx xxxx

IMAP_HOST=imap.gmail.com
IMAP_PORT=993
IMAP_USER=your@gmail.com
IMAP_PASS=xxxx xxxx xxxx xxxx

# AI
ANTHROPIC_API_KEY=sk-ant-api03-...
```

---

## crm-backend

```env
# Server
PORT=4000
DATABASE_URL=postgresql://user:pass@host:5432/twilio_app
JWT_SECRET=your_jwt_secret_here
NODE_ENV=production

# CORS — comma-separated
ALLOWED_ORIGINS=https://finance-service-three.vercel.app,https://crm-service-next.vercel.app
```

---

## finance-service

```env
# CRM backend URL — used by Vite proxy in dev
VITE_CRM_API_URL=http://localhost:4000

# Navigation link to CRM app
VITE_CRM_URL=https://crm-service-next.vercel.app
```

**Production (Vercel):** `vercel.json` rewrites `/api/:path*` to `https://crm-backend-tlp2.onrender.com/api/:path*` — no env var needed in prod.

---

## Shared Variables (must match across services)

| Variable | Shared By | Why It Must Match |
|----------|-----------|-------------------|
| `DATABASE_URL` | All backends | Same PostgreSQL database |
| `JWT_SECRET` | twilio-voice-service, crm-service-next, crm-backend | Same users table, tokens must be verifiable by any service |

---

## Local Dev Setup Checklist

| Service | Command | Port |
|---------|---------|------|
| PostgreSQL | Start your local Postgres | 5432 |
| [[twilio-voice-service]] | `npm start` or `node server.js` | 3000 |
| [[crm-service-next]] | `npm run dev` | 4000 |
| [[crm-backend]] | `node server.js` | 4000 (conflict!) |
| [[calling_application_fe]] | `npm run dev` | 5173 |
| [[finance-service]] | `npm run dev` | 3001 |
| ngrok (for Twilio) | `ngrok http 3000` | — |

**Port 4000 conflict:** crm-service-next and crm-backend cannot both run on 4000 locally. Options:
- Change crm-backend to port 4001 locally
- Only run the one you need at a time
- Use Docker Compose with different ports

---

## Deployment — Production URLs

| Service | Production URL | Platform |
|---------|---------------|----------|
| [[twilio-voice-service]] | Custom / Render | Render / Docker |
| [[calling_application_fe]] | Vercel | Vercel |
| [[crm-service-next]] | crm-service-next.vercel.app | Vercel |
| [[crm-backend]] | crm-backend-tlp2.onrender.com | Render (Singapore) |
| [[finance-service]] | finance-service-three.vercel.app | Vercel |
