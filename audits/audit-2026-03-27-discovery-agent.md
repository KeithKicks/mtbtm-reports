# Discovery Agent Weekly Audit Report
**Date:** 2026-03-27 | **Auditor:** Claude (automated) | **Period:** Week of 2026-03-21 to 2026-03-27

---

## Executive Summary

The Discovery Agent (v0.7.0) has seen rapid development -- 7 releases in 4 days (0.1.0 through 0.7.0, March 24-27). The architecture is solid: TypeScript/Express, SQLite persistence, structured PSD schema with a 13-check scoring rubric, and a well-designed 5-phase conversational flow. However, the service is **not currently running**, has **no environment configured**, and has several security and resilience gaps that need attention before production use.

---

## System Overview

| Attribute | Value |
|-----------|-------|
| Location | `/Users/keithkicks/discovery-agent/` |
| Runtime | Node 20, TypeScript, Express 4 |
| Port | 3001 |
| Version | 0.7.0 |
| DB | better-sqlite3 (`data/discovery.db`) |
| LLM | Claude Sonnet 4.6 (chat/PSD), Claude Opus 4.6 (reasoning) |
| Cost per session | ~$3 (actual Claude API spend) |
| Budget | $10 default |
| Deploy target | Railway |
| Tests | 31 across 4 files |
| Status | **NOT RUNNING** |

---

## Findings

### CRITICAL: Service Not Running / Not Configured

The Discovery Agent is completely offline:
- Port 3001 is not in use
- `node_modules/` not installed (`npm ci` required)
- No `.env` file exists (no API key configured)
- No `data/` directory (no persistent database)
- No `.env.example` template to guide setup

**Impact:** The MTBTM Machine's `poll-discovery` endpoint hardcodes `http://localhost:3001/api/projects`. With the service down, the entire discovery-to-pipeline bridge is non-functional. New project intake via the Discovery Agent path is impossible.

**Suggestion:** Create a `.env.example` with required vars. Document the startup sequence. Consider a health check script that verifies the service is reachable.

---

### CRITICAL: In-Memory Session Loss on Restart

The `SessionManager` stores all active discovery sessions in a JavaScript `Map`. SQLite (`ContextStore`) only persists completed projects and artifacts -- **not in-progress conversations**.

If the service restarts mid-conversation, all active sessions are permanently lost. Given Railway's `ON_FAILURE: restart` policy, this could happen at any time.

**Impact:** A user could be 15 minutes into a discovery conversation and lose everything on a service restart.

**Suggestion:** Persist session state to SQLite on every message. The `ContextStore` already has a `messages` table -- extend it to store in-progress session state and restore on startup.

---

### HIGH: No Authentication

The Express API has zero auth. Any HTTP client can:
- Create sessions
- Generate PSDs
- Approve projects (writes to disk)
- List all projects and their PSDs

Combined with the open CORS policy (`app.use(cors())`), the service is fully accessible from any origin.

**Suggestion:** Add API key auth (simple header check like the Machine's `API_KEY` env var pattern). Add CORS origin restrictions matching the Machine's approach.

---

### HIGH: PSD Schema Mismatch with IntakeAgent

Two paths produce PSDs for the Machine:

| Field | Discovery Agent PSD | IntakeAgent PSD |
|-------|-------------------|-----------------|
| Acceptance criteria | Structured objects (`{given, when, then}`) | Simple string arrays |
| Success criteria | Full objects with IDs, linked to features | Not present |
| Risk register | Full objects with likelihood/impact/mitigation | Simpler structure |
| System config | Full config block | Not present |

The `PSDDecomposerAgent` handles both via loose dict parsing, but this is fragile and untested for edge cases.

**Suggestion:** Define a canonical PSD interface that both producers conform to, or add explicit adapter logic in the decomposer with test coverage for both formats.

---

### HIGH: Hardcoded Discovery URL in Machine

The Machine's `poll_discovery` endpoint has `http://localhost:3001/api/projects` hardcoded. No env var, no fallback, no timeout handling.

**Suggestion:** Move to an env var (`DISCOVERY_AGENT_URL`). Add a timeout and graceful error handling so the Machine doesn't hang if the Discovery Agent is unreachable.

---

### MEDIUM: No Rate Limiting

The Express server has no rate limiting middleware. While the internal `LLMClient` throttles to 1.5s between Claude calls, the HTTP endpoints are unprotected. A client could:
- Spam session creation
- Trigger many concurrent LLM calls
- Fill the disk via PSD approval writes

**Suggestion:** Add `express-rate-limit` middleware. The Machine already has rate limiting as a pattern to follow.

---

### MEDIUM: No Test Coverage for File Attachments

The attachment system (`utils/attachments.ts`, `types/attachments.ts`) supports up to 5 files, 10MB each, across 8 MIME types. But there are **zero tests** for attachment validation or the upload flow.

**Suggestion:** Add unit tests for attachment validation (size limits, MIME type filtering, count limits) and integration tests for the message endpoint with file uploads.

---

### MEDIUM: No Test Coverage for poll-discovery Bridge

The Machine-side `GET /api/pipeline/poll-discovery` endpoint has zero test coverage. It makes a live HTTP call to `localhost:3001`, which means:
- Tests would need a mock server or httpx mock
- The current test suite can't catch regressions in the import flow

**Suggestion:** Add tests using `respx` or `httpx` mock transport to simulate Discovery Agent responses.

---

### MEDIUM: Monolithic Frontend

`public/index.html` is 22K+ tokens of inline React/Babel. No build step, no component splitting, no CSS framework. This was appropriate for rapid prototyping but will become painful to maintain.

**Suggestion:** Not urgent, but plan a migration to a proper React build (Vite + React) when the UI needs its next significant feature.

---

### LOW: tsx Runtime Workaround in Docker

The Dockerfile installs `tsx` separately after `npm ci --omit=dev` because `tsx` is in `devDependencies` but needed at runtime. This is a footgun that could break on dependency updates.

**Suggestion:** Move `tsx` to `dependencies`, or better yet, compile TypeScript at build time and run the JS output directly with Node.

---

### LOW: Phase Readiness is Keyword-Based

Phase advancement uses keyword pattern matching on conversation history:
- Phase 1: checks for "problem", "user" keywords in 2+ messages
- Phase 2: checks for "risk", "constraints", "technology"
- Phase 3: checks for "features", "quality"

This could pass prematurely if the user mentions these words in passing, or block if the user describes the same concepts with different vocabulary.

**Suggestion:** Consider LLM-based phase readiness checks for higher accuracy (tradeoff: latency + cost per check).

---

### LOW: goToPhase Only Goes Backward

The `goToPhase` method enforces `targetIdx < currentIdx` -- you can only jump back, never forward. This is intentional but not communicated to the user.

**Suggestion:** Add a clear message in the UI/response when a forward jump is attempted.

---

## PSD Quality Assessment

The 13-check scoring rubric is well-designed:
- **100 points total** across 5 categories (success criteria, features, traceability, risks, completeness)
- **70/100 threshold** is reasonable
- **Force-approve available** as an escape hatch
- Heaviest weights on structured acceptance criteria (15) and success metrics (15) -- good priorities

The rubric has full test coverage (10 tests covering all 13 checks).

---

## Architecture Assessment

**Strengths:**
- Clean separation: agents / core / middleware / types / utils
- Zod schemas for all data structures (PSD, config, messages)
- LLM client with built-in throttling and retry
- SQLite for persistence, in-memory sessions for active conversations
- Structured logging throughout
- Railway deployment config ready

**Weaknesses:**
- Session volatility (in-memory only)
- No auth layer
- No rate limiting
- Tight coupling to localhost:3001 from Machine side
- Two PSD schema dialects (Discovery vs IntakeAgent)

---

## Top 5 Recommendations (Priority Order)

1. **Get the service running** -- Install deps, create `.env`, start the service. The entire discovery pipeline is blocked without it.

2. **Persist sessions to SQLite** -- A restart should not lose user conversations. This is the highest-impact reliability fix.

3. **Add authentication** -- Even a simple API key. The service writes to disk and calls Claude (~$3/session) on unauthenticated requests.

4. **Externalize the discovery URL** -- Replace the hardcoded `localhost:3001` in the Machine with an env var. Add timeout + error handling.

5. **Unify PSD schema** -- Define one canonical PSD format. Add adapter tests. This prevents silent breakage as both producers evolve independently.

---

## Release Velocity

| Version | Date | Key Changes |
|---------|------|-------------|
| 0.1.0 | 2026-03-24 | Initial Discovery Agent |
| 0.2.0 | 2026-03-24 | Structured PSD + scoring |
| 0.3.0 | 2026-03-25 | Session management + phases |
| 0.4.0 | 2026-03-25 | Pipeline states + approval |
| 0.5.0 | 2026-03-25 | File attachments |
| 0.6.0 | 2026-03-26 | Go-back/go-to-phase nav |
| 0.7.0 | 2026-03-27 | Config schema + system settings |

7 releases in 4 days indicates active development. The changelog is well-maintained but the pace suggests the codebase may benefit from a stabilization pass before adding more features.
