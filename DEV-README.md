# ITeams — Developer Overview

**Version:** 1.0.2.1S (Security)

---

## Version History

| Version | Code | Focus |
|---|---|---|
| 1.0.0.0S | Stable | Initial production release |
| 1.0.1.0S | Stable | Theme hardening, light mode fixes |
| 1.0.2.0S | Stable | Focus mode pipeline, SQL auto-repair, rate limits |
| 1.0.2.1S | Security | Helmet CSP, login rate limit, SQL injection sanitization |

---

## What This Software Does

A browser-based internal operations platform with an AI agent that can query databases, run reports, manage subscribers, handle billing, and automate tasks through natural language.

---

## Modules & Features

### Dashboard
- Role-based homepage with summary cards
- Quick-action shortcuts

### Tasks
- Full CRUD task board
- Status workflow (todo → in_progress → review → done → cancelled)
- Priority levels, assignments, due dates
- Comments, overlays, custom views
- Bulk operations

### Messaging
- Direct messages
- Group conversations
- Real-time WebSocket delivery

### Integrations — Accounts
- Subscriber ledger viewer (dr/cr transaction history)
- Balance calculation per user
- Bill payment recording
- Payment gateway pending approvals
- User profile editing
- Transaction search and filtering
- PDF invoice printing

### Integrations — Radius
- ISP subscriber list with search and filtering
- Expiration date management
- Account enable/disable
- Bulk operations
- Expiry extension with notes

### Integrations — Near Expiration
- Filtered list of subscribers expiring within configurable days
- Bulk extension workflow

### Integrations — Unpaid Bills
- Users with positive ledger balance
- Transaction drill-down per user
- Search by username, phone, address

### Integrations — Ledger
- Cross-user ledger viewer
- Manual transaction entry (dr/cr)
- Transaction editing and deletion
- Search across all transactions

### Integrations — SMS
- Manual SMS sending
- Billing reminder templates
- Expiry warning automation
- Delivery status tracking

### Integrations — Audit Logger
- Read-only audit trail of all mutations
- JSON payload inspection
- Entity table mapping for restore previews

### Integrations — Custom Views
- Saved SQL queries as interactive pages
- Parameterisation, column sorting, pagination

### Billing / GST (Postgres-based)
- Company setup with multi-currency
- Customer management
- Item master (SKU, tax, HSN/SAC)
- Price lists
- Inventory locations and stock transfers
- Documents (invoices, credit notes, purchases, receipts)
- GST compliance (HSN/SAC lookup, ITC ledger)
- Payment tracking
- E-way bill support
- SMS automation for GST invoices
- Activity logs

### User & Role Management
- Create/edit/delete users
- Role-based access control with granular module permissions
- Per-role toggle for every integration, task action, and communication feature
- Owner and manager roles with escalation paths

### Tau — AI Agent
- Multi-turn conversation with tool execution
- Natural language to SQL
- Automatic schema discovery via LE GeX knowledge graph
- Read-only SQL execution against PostgreSQL with 15s timeout
- Subscriber search and ledger lookup tools
- Payment request listing
- Task overlay management
- Custom view saving
- Math evaluation
- Multi-provider model routing (Kilo Code primary, Gemini/OpenRouter fallback)
- Focus Mode for SQL/reporting accuracy
- Forced reasoning mode
- Agent mode with tool budget and multi-step planning
- Chat history with thread management
- Auto-thread titling
- File attachments (CSV, XLSX, PDF, images with OCR, DOCX, text)

### Tau — Admin Settings
- AI provider selection (Kilo, Gemini, OpenRouter)
- Model picker with catalog (40+ models)
- Custom system instructions
- Agent creation wizard
- Action history with undo
- Workflow builder (multi-step automations)
- Macro recorder
- Document ingestion and semantic query

### Search
- Global workspace search
- Redex command bar
- LE GeX contextual search

---

## Quick Reference

### Keyboard Shortcuts

| Shortcut | Action |
|---|---|
| Ctrl+E | Toggle simplE mode |
| Ctrl+M | Toggle mobile layout |
| Ctrl+D | Toggle Diorite mode |
| Ctrl+L | Logout |
| Ctrl+R | Reset layout to defaults |
| Ctrl+U | Show What's New splash |

### Focus Mode Pipeline (when activated)

1. Task classification + schema pruning
2. Ambiguity probe (detect domain term collisions)
3. Force plan generation
4. Multi-path SQL execution (2-3 variants, cross-check results)
5. SQL validation checklist
6. Output verification
7. Structured answer with evidence

### Rate Limits

| Endpoint | Limit | Scope |
|---|---|---|
| `/auth/login` | 10 / 15 min | Per IP |
| `/chat`, `/chat/stream` | 30 / 60 s | Per session |
| All AI requests | 200 / day | Per user (configurable) |

---

## Dependencies (Runtime)

| Layer | Key Packages |
|---|---|
| Runtime | Node.js 24, Express, TypeScript |
| Frontend | React 19, Vite, Zustand, Radix UI, Lucide icons |
| Database | pg, mysql2 |
| Auth | bcryptjs, jsonwebtoken |
| AI | tsx (TypeScript runner), Zod validation |
| Security | helmet, express-rate-limit |
| Files | multer, xlsx, pdfjs-dist, mammoth, tesseract.js |

---

## Architecture (Top View)

```
Browser (React SPA)
  │
  ├── Apache (reverse proxy, port 443)
  │     └── /api/* → Express backend (port 4000)
  │
  ├── Express (API server, systemd-managed)
  │     ├── PostgreSQL 16 (app state, AI, billing)
  │     └── MySQL 8 (legacy subscriber/ledger mirror)
  │
  └── Docker
        └── PostgreSQL 16 container (port 55435)
```

### Data Flow

- **Application data** → PostgreSQL (primary, managed by app ORM)
- **Legacy subscriber/ledger data** → MySQL → periodic sync → PostgreSQL `legacy.*` schema
- **Writes through ITeams UI** → MySQL + synchronous per-row mirror to PostgreSQL
- **AI chat** → Express → LLM provider → tool execution → response stream

---

## Configuration (Environment)

All configuration lives in `server/src/config.ts`, sourced from environment variables. Key items:

| Variable | Purpose |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `MYSQL_*` | Legacy MySQL credentials |
| `KILO_API_KEY` | Primary AI provider |
| `GEMINI_API_KEY` | Fallback AI provider |
| `OPENROUTER_API_KEY` | Secondary fallback |
| `JWT_SECRET` | Token signing (must be custom in production) |
| `CORS_ORIGIN` | Allowed frontend origins |
| `PORT` | Express listen port |
| `LEGACY_CACHE_INTERVAL_MS` | MySQL sync polling frequency |
