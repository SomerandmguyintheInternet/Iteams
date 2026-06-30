# ITeams — Internal Operations Platform

A production-grade internal operations platform evolved across six major versions, transitioning from a lightweight billing utility into an AI-native business execution environment powered by Tau.

---

## Architecture Overview

ITeams is a full-stack application with a React (Vite) frontend and an Express/Node.js backend, connected to PostgreSQL with a mirrored MySQL legacy database for ISP/Radius subscriber data.

| Layer | Technology |
|---|---|
| Frontend | React 19, Vite, TypeScript, Zustand, Radix UI |
| Backend | Express, TypeScript, tsx (TypeScript runtime) |
| Database | PostgreSQL 16 (primary), MySQL 8 (legacy mirror) |
| AI | Multi-provider (Kilo Code, Gemini, OpenRouter) |
| Deployment | Docker, systemd, Apache reverse proxy |

---

## V1 — The Lightweight Utility Era

The initial version was designed as a fast internal operational tool — not an ERP, not an enterprise suite. The philosophy was "do the job and get out."

**Core features:** Customer lookup, billing operations, item management, transaction entry, basic reports, search, authentication.

**Architecture:** Very small codebase with minimal dependencies. Simple pages with direct workflows and few abstractions. Fast navigation was the primary design goal.

**Database:** Single MySQL instance for all operational data.

---

## V2 — Workflow Expansion

Reduced operational friction by expanding workflows without expanding architecture.

**Added:** Better reports, improved customer pages, enhanced search, more billing screens, better transaction handling, improved filtering, better document handling.

**Architecture:** Still lightweight. No significant architectural changes — iterative improvements to the existing patterns.

---

## V3 — Operational Maturity

The application became a serious daily-use tool.

**Major additions:**
- Reporting improvements (better ledgers, totals, filters, exports)
- Billing improvements (faster workflows, better payment handling, better customer tracking)
- UI improvements (cleaner pages, better navigation, better responsiveness)
- Backend improvements (cleaner APIs, better organization, reduced duplication)

**Architecture:** The codebase began stabilizing. API structure formalized. Common patterns emerged for data access, authentication, and routing.

---

## V4 — Intelligence Introduction

The first major architectural shift. The application remained relatively lightweight, but Tau was introduced — a Q&A assistant that could help with business questions, provide context-aware answers, and assist with daily operations.

**Architecture:** Introduction of an AI layer alongside the existing REST architecture. The system prompt approach was used rather than fine-tuning, allowing the AI to understand the database schema and business rules through injected context.

**Key architectural addition:** AI routing layer between user requests and model responses. System prompts became the primary mechanism for AI behavior control.

**V4.1 Ultimate** — The final refined version of the classical architecture:
- Better context handling and memory within sessions
- More grounded answers with less hallucination
- Database awareness, report awareness, billing awareness
- Faster responses with better chat UI

---

## V5 — Platform Consolidation

Everything got cleaner without an AI revolution. The application was still intentionally lightweight.

**Major features:**
- Dedicated billing report pages with better navigation
- Near-expiration customer tracking
- Unpaid customer workflows
- Better document handling
- Better SMS integration and external APIs
- Better upload systems

**Tau during V5:** Incremental improvements to answers, speed, and integration — still fundamentally a chat assistant rather than a business execution engine.

**Architecture:** More polished APIs, better internal organization, better maintainability, better modularity. The codebase was restructured for clarity without adding complexity.

---

## V6 — AI-Native Operating Layer

The breakpoint where Tau stopped being a feature and became a platform capability.

### Tau AI Engine

The AI system operates as a multi-step agent with tool execution, planning, and verification:

**Provider abstraction layer** — Routes requests across Kilo Code (primary), Gemini, and OpenRouter based on availability, rate limits, and task type. Auto-model-picking selects the optimal model per request.

**Focus Mode** — A strict execution pipeline activated for SQL/reporting tasks:
1. Task classification and schema pruning
2. Ambiguity probing for domain terms
3. Force planning before tool calls
4. Multi-path SQL execution (2-3 structurally different queries cross-checked)
5. SQL validation (columns, JOINs, GROUP BY, aggregates, timeout)
6. Output verification (row count, value plausibility, zero-result sanity)
7. Structured answer format

**Automatic SQL repair** — When a query fails with "column does not exist," the system extracts the column name, queries `information_schema.columns`, finds the closest match via Levenshtein distance, rewrites the query, and retries — all before the AI sees the error.

**Tools:** `execute_sql` (read-only with forbidden keyword guard and 15s timeout), `search_subscribers`, `get_subscriber_ledger`, `list_payment_requests`, `save_custom_view`, `search_ai_memory`, `evaluate_math`, `bump_subscriber_expiry`, `extend_expiry`.

### Rate Limiting & Security

Three-tier rate limiting:
- **Per-second:** 30 requests per 60 seconds via `express-rate-limit` on `/chat` and `/chat/stream`
- **Per-day:** Hard cap of 200 requests/day for all non-owner users (configurable via ResTFuLL permissions)
- **Auth:** 10 login attempts per 15 minutes per IP

### Security Measures

- bcryptjs password hashing with cost factor 10
- Helmet security headers (CSP disabled for React SPA compatibility)
- Parameterized PostgreSQL queries via `$1` bind parameters
- JWT authentication with httpOnly cookies + Bearer header dual submission
- SameSite=Lax cookie attribute for CSRF mitigation
- JWT secret validation preventing production startup with default secret
- CORS whitelist restricting origins
- X-Powered-By header disabled

### Database Architecture

**PostgreSQL (primary):** Application state, chat threads, AI messages, user profiles, roles, permissions, billing documents (GST/B2B), tasks, todos, messages, documents (osdock/oslink), workflow definitions, AI document embeddings.

**MySQL (legacy mirror):** ISP subscriber data (rm_users), accounts ledger (acc_lazor), payment gateway (payment_requests). Synced to PostgreSQL via `legacy.*` schema tables using an adaptive polling mechanism.

**Schema sync:** The legacy cache layer polls MySQL at configurable intervals (10s active, 60min idle), detects changes via fingerprint (row count + MAX(txn_id)), and upserts into PostgreSQL mirror tables. Per-row mirror functions (`mirrorAccLazorTxn`) provide synchronous write-through on ITeams-originated mutations.

### Frontend Architecture

**State management:** Zustand stores for auth, settings, AI state, billing, voice, chat, presence, diorite mode, and legacy mirror generation.

**UI components:** Radix UI Dialog primitives wrapped in a custom `Modal` component with `forceFloating` support for centered dialogs. CSS module system for component-scoped styles with CSS custom properties for theming.

**CSS design system:** Design tokens (bg, fg, line, accent, shadow, glass) defined as CSS custom properties on `:root` with `html[data-theme="dark"]` overrides. Light/dark theme switching via `data-theme` attribute on `<html>`.

**Navigation:** CSS Grid-based app shell (`app-shell`) with sidebar (248px), toolbar, and scrollable content area (`page-scroll`). Supports four nav positions: top, bottom, left, right. Mobile mode collapses sidebar to bottom tab bar.

### Theming System

- `data-theme="dark"` / `data-theme="light"` on `<html>`
- `:root` defines light-mode defaults, `html[data-theme="dark"]` overrides for dark mode
- Tau UI uses `--tau-*` CSS variables defined in `globals.css` with fallback values for resilience
- Smooth transitions between themes via CSS variable resolution at computed-value time

### Deployment

- Systemd service (`iteams-v6`) with automatic restart on failure
- Docker PostgreSQL with `--restart always`
- Apache reverse proxy routing `/api/` to the Express backend
- Port 4000 for the application, port 55435 for the dedicated PostgreSQL instance
- Environment variables configured in the systemd service file
