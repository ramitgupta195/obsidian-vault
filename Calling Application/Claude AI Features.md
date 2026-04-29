---
tags: [calling-app, integration, ai, claude, anthropic]
---

# Claude AI Features

> [[00 - Index|← Back to Index]]

## Overview

Both [[twilio-voice-service]] and [[crm-service-next]] use the **Anthropic Claude API** for AI-powered features. They use different Claude models depending on the complexity of the task.

---

## All AI Features — Master Table

| Feature | Service | Model | Max Tokens | Output |
|---------|---------|-------|-----------|--------|
| Lead scoring | [[twilio-voice-service]] | claude-sonnet-4-6 | — | Score 0-100 + reasoning |
| Lead scoring | [[crm-service-next]] | claude-opus-4-5 | 500 | Score 0-100 + reasoning |
| Next-best-action | [[twilio-voice-service]] | claude-sonnet-4-6 | — | Action list |
| Next actions (saved) | [[crm-service-next]] | claude-sonnet-4-6 | 1000 | 3-4 actions saved to DB |
| Deep client analysis | [[crm-service-next]] | claude-opus-4-5 | 1500 | Summary, risk factors, opportunity score |
| Email draft | [[twilio-voice-service]] | claude-sonnet-4-6 | — | Subject + body |
| Email draft | [[crm-service-next]] | claude-sonnet-4-6 | 800 | Subject + body |
| Pipeline forecast | [[twilio-voice-service]] | claude-sonnet-4-6 | — | Revenue forecast |
| Call summarization | [[crm-service-next]] | claude-haiku-4-5 | 400 | 3-5 bullet points |
| Course suggestion (live) | [[twilio-voice-service]] | claude-sonnet-4-6 | — | Relevant courses |

---

## Model Selection Strategy

```mermaid
graph TD
    A{Task type} --> B[Simple / fast\ne.g. call summary]
    A --> C[Moderate\ne.g. email draft, next actions]
    A --> D[Complex / deep\ne.g. full client analysis]

    B --> E[claude-haiku-4-5\nFastest, cheapest]
    C --> F[claude-sonnet-4-6\nBalanced]
    D --> G[claude-opus-4-5\nMost capable]
```

---

## Lead Scoring

**Endpoint:** `POST /api/ai/lead-score`
**Model:** `claude-opus-4-5` | **Max tokens:** 500

**Input signals:**
- Lead status (new / contacted / qualified / converted / disqualified)
- Lead type (b2b / b2c)
- Number of calls made
- Number of notes logged
- Number of open deals
- Number of course interests

**Output (JSON):**
```json
{
  "score": 72,
  "reasoning": "Lead has been contacted 3 times, has 2 open deals..."
}
```

Score stored in `crm_clients.ai_score` for display in UI.

---

## Next Actions (crm-service-next)

**Endpoint:** `GET /api/ai/next-actions/[id]`
**Model:** `claude-sonnet-4-6` | **Max tokens:** 1000

Context built from: full client data — courses, deals, calls, notes, tasks via `buildClientContext()` in `lib/aiHelpers.js`

**Output (JSON array saved to DB):**
```json
[
  {
    "action_type": "call",
    "reasoning": "Client has not been contacted in 14 days",
    "priority": "high"
  },
  {
    "action_type": "email",
    "reasoning": "Send ITIL course proposal based on their interest",
    "priority": "medium"
  }
]
```

**Action types:** call, email, follow_up, meeting, proposal, renewal, upsell

**Persistence:** Results saved to `crm_ai_suggestions` table. Auto-deleted on each regeneration — always fresh.

---

## Deep Client Analysis

**Endpoint:** `POST /api/ai/analyze-client`
**Model:** `claude-opus-4-5` | **Max tokens:** 1500

**Output:**
```json
{
  "analysis": "This client is a high-value B2B account...",
  "next_steps": ["Schedule renewal call", "Propose PRINCE2 course"],
  "risk_factors": ["Last contact was 30 days ago", "Deal stalled"],
  "opportunity_score": 85
}
```

---

## Email Drafting

**Endpoint:** `POST /api/ai/email-draft`
**Input:** `{ clientId, purpose, additionalContext }`
**Model:** `claude-sonnet-4-6` | **Max tokens:** 800

**Output:**
```json
{
  "subject": "Following up on your ITIL 4 interest",
  "body": "Dear [Name],\n\nFollowing our call last week..."
}
```

Agent reviews, edits, then sends via the `/mail` page.

---

## Call Summarization

**Endpoint:** `POST /api/ai/summarize-call/[id]`
**Model:** `claude-haiku-4-5-20251001` (fastest/cheapest)
**Max tokens:** 400

Input: raw call transcript from `call_transcriptions` table
Output: 3-5 bullet points stored back in `call_logs.ai_summary`

Uses Haiku intentionally — summaries are simple and high volume.

---

## Live Course Suggestion (during calls)

**Endpoint:** `POST /api/knowledge/suggest`
**Triggered by:** [[calling_application_fe]] detecting keywords in live transcript

**Keyword examples:** agile, scrum, itil, devops, six sigma, pmp, prince2, cissp, cobit

**Flow:**
```
Transcript chunk arrives
    → FE scans for keywords
    → POST /api/knowledge/suggest { keywords, call_sid }
    → AI matches + ranks courses
    → Displayed in "Course Suggestions" panel in Dialer
```

---

## Duplication Note

Both [[twilio-voice-service]] and [[crm-service-next]] have **overlapping AI features** (lead scoring, email drafts, next actions). This is because the two CRM modules were built independently.

In a merged app (see [[Merge Strategy]]), there would be a single AI service layer.

---

## Connects To

- [[twilio-voice-service]] — AI in calling/CRM context
- [[crm-service-next]] — AI in CRM context
- [[Call Lifecycle]] — course suggestion during live calls
- [[Merge Strategy]] — consolidation plan
