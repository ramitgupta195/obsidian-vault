---
tags: [calling-app, index, architecture]
---

# Calling Application — Documentation Hub

> Full system KT for the Invensis Learning calling + CRM + finance platform.

## What is this system?

A multi-service platform that lets sales agents make browser-based calls via Twilio, manage leads/deals/clients in a CRM, track finances, and get AI-powered insights — all backed by a shared PostgreSQL database.

---

## Services

| Service | Type | Purpose |
|---------|------|---------|
| [[twilio-voice-service]] | Express Backend | Calling engine — Twilio, CRM, billing, AI |
| [[calling_application_fe]] | React Frontend | Main agent UI for making/receiving calls |
| [[crm-service-next]] | Next.js Full-Stack | CRM platform — clients, leads, deals, email |
| [[crm-backend]] | Express Backend | Finance & accounting API |
| [[finance-service]] | React Frontend | Finance UI — invoices, P&L, expenses |

---

## Architecture & Flows

- [[High-Level Architecture]] — Full system diagram
- [[Service Communication Map]] — Who calls who and at what URL
- [[Auth Flow]] — How JWT auth works across all services
- [[Call Lifecycle]] — End-to-end call flow (dial → transcription → billing → ACW)
- [[Database Schema]] — All tables, ownership, relationships

---

## Integrations

- [[Twilio Integration]] — Calling, conferences, webhooks, supervisor controls
- [[Claude AI Features]] — All AI features across services
- [[Gmail Integration]] — SMTP send + IMAP sync

---

## Reference

- [[Environment Variables]] — All env vars for every service
- [[Merge Strategy]] — How to unify all 5 repos into one app
- [[Deployment Setup]] — Current ports, URLs, platforms

---

## Graph Key

In the graph view, each node is a service or topic. Links represent real dependencies — if Service A calls Service B, there is a wikilink between them.
