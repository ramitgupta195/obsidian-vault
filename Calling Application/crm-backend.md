---
tags: [calling-app, service, backend, finance]
---

# crm-backend

> [[00 - Index|← Back to Index]]  
> **Type:** Node.js / Express Backend  
> **Port:** 4000  
> **Deployed:** Render (Singapore region)  

---

## What It Does

A **dedicated finance & accounting API backend**. It has no frontend — it purely serves:
1. The [[finance-service]] React frontend (all finance UI calls go here)
2. [[crm-service-next]] for its `/api/finance/*` routes

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Runtime | Node.js (ES Modules) |
| Framework | Express.js 4.19 |
| Database | PostgreSQL via `pg` driver (raw SQL) |
| Auth | JWT (HS256) + bcryptjs |
| Deploy | Render (Singapore) — free tier |

---

## All API Endpoints

### Auth
| Method | Path | Purpose |
|--------|------|---------|
| POST | `/api/auth/login` | Login → JWT |
| GET | `/api/auth/me` | Current user |
| GET | `/health` | Health check (no auth) |

### Invoices
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/api/finance/invoices` | List (filter: status, search, limit) |
| GET | `/api/finance/invoices/:id` | Detail |
| POST | `/api/finance/invoices` | Create (auto-status: draft) |
| PUT | `/api/finance/invoices/:id` | Update (auto-recalculates status) |
| DELETE | `/api/finance/invoices/:id` | Delete |

**Invoice status logic:**
- `paid_amount = total_amount` → status: `paid`
- `0 < paid_amount < total_amount` → status: `partial`
- Otherwise → `draft` or `sent`

### Payments
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/api/finance/payments` | List (max 200, default 100) |
| POST | `/api/finance/payments` | Record payment — uses `SELECT FOR UPDATE` on invoice |

### Expenses
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/api/finance/expenses` | List (filter: category, status, search) |
| POST | `/api/finance/expenses` | Create (auto-status: pending) |
| PUT | `/api/finance/expenses/:id` | Update / approve / reject |

**Categories:** trainer_fee, software, venue, travel, marketing, other

### Vendors
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/api/finance/vendors` | List all |
| POST | `/api/finance/vendors` | Create |
| PATCH | `/api/finance/vendors/:id` | Update |

### Bank Accounts
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/api/finance/bank-accounts` | List (sorted: region, default first) |
| POST | `/api/finance/bank-accounts` | Create (transaction: auto-clear is_default for region) |
| PUT | `/api/finance/bank-accounts/:id` | Update (handles is_default logic) |

**Regions:** india, us, uk, europe, singapore, australia

### Quotes
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/api/finance/quotes` | List |
| POST | `/api/finance/quotes` | Create (auto-number: QT-YYYY-####) |
| PUT | `/api/finance/quotes/:id` | Update — can convert accepted quote to invoice in one transaction |
| DELETE | `/api/finance/quotes/:id` | Delete (draft only) |

### Credit Notes
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/api/finance/credit-notes` | List |
| POST | `/api/finance/credit-notes` | Create (auto-number: CN-YYYY-####) |
| PUT | `/api/finance/credit-notes/:id` | State machine update |

**State machine:** `draft → submitted → approved → processed`
If type = `offset` and processed → reverses invoice payment automatically.

### Purchase Orders
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/api/finance/purchase-orders` | List |
| POST | `/api/finance/purchase-orders` | Create (auto-number: PO-YYYY-####) |
| PUT | `/api/finance/purchase-orders/:id` | Update |
| DELETE | `/api/finance/purchase-orders/:id` | Delete (draft only) |

### Reporting
| Method | Path | Purpose |
|--------|------|---------|
| GET | `/api/finance/dashboard` | KPIs: collected, outstanding, expenses, overdue |
| GET | `/api/finance/cashflow` | 6-month view, burn rate, runway months |
| GET | `/api/finance/pl` | P&L: revenue/COGS/OpEx, margins, 6-month trend |
| GET | `/api/finance/tax` | GST (18% on INR) + TDS (10% on trainer fees, Sec 194J) |

---

## Auto-Numbering

All financial documents get sequential IDs:

| Document        | Format        | Example       |
| --------------- | ------------- | ------------- |
| Invoices        | INV-YYYY-#### | INV-2026-0042 |
| Quotes          | QT-YYYY-####  | QT-2026-0007  |
| Credit Notes    | CN-YYYY-####  | CN-2026-0003  |
| Purchase Orders | PO-YYYY-####  | PO-2026-0015  |

---

## Transaction Safety

Uses PostgreSQL transactions with `SELECT FOR UPDATE` for:
- Recording payments (prevents double-payment)
- Setting default bank account (clears others in region atomically)
- Quote-to-invoice conversion
- Credit note offset processing

---

## Multi-Region Bank Accounts

| Region | Currency | Tax |
|--------|----------|-----|
| India | INR ₹ | GST 18% |
| US | USD $ | Zero-rated |
| UK | GBP £ | VAT 20% |
| Europe | EUR € | VAT 19% |
| Singapore | SGD S$ | GST 9% |
| Australia | AUD A$ | GST 10% |

---

## Auth Middleware

```javascript
requireAuth:
  - Extract Bearer token from Authorization header
  - Verify JWT with JWT_SECRET
  - Attach req.user = { id, email, role, name }
  - Returns 401 if missing or invalid

requireAdmin:
  - Chains requireAuth
  - Checks req.user.role === "admin"
  - Returns 403 if not admin
```

---

## Port Conflict Note

⚠️ Both [[crm-service-next]] (Next.js) and `crm-backend` (Express) use **port 4000** locally. In production they are on different URLs:
- crm-service-next → Vercel
- crm-backend → Render (`crm-backend-tlp2.onrender.com`)

---

## Connects To

- [[finance-service]] — serves this frontend
- [[crm-service-next]] — finance routes overlap
- [[Database Schema]] — finance tables (finance_*)
- [[Auth Flow]] — JWT auth details
- [[Environment Variables]] — config
