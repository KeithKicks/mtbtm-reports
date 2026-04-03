# Discovery Agent Audit Report
**Date:** 2026-04-03 | **Auditor:** Claude (automated) | **Period:** 2026-03-30 to 2026-04-03

---

## Executive Summary

Major iteration day. The Discovery Agent went from v1.0.0 to v1.3.0 in a single session with three focused releases: markdown PSD output (replacing fragile JSON), stakeholder alignment features (pause memos, gap tracking, friendlier tone), and a complete UI redesign (light theme, 3-column layout). All 34 tests passing. Live on Railway.

---

## System Overview

| Attribute | Value |
|-----------|-------|
| Version | **1.3.0** (was 1.0.0) |
| Runtime | Node 20, TypeScript, Express 4 |
| Port | 3001 |
| DB | better-sqlite3, WAL mode |
| LLM | Claude Sonnet 4.6 (chat + PSD generation) |
| Tests | **34 passing** (4 suites) |
| Deploy | Railway (auto-deploy from GitHub) |
| Status | **Live** |

---

## What Changed (3 releases)

### v1.1.0 — Markdown PSD
The biggest architectural change since v1.0. PSD generation switched from forcing Claude to output a 22-field nested JSON object via tool_use (which caused frequent Zod validation failures) to a two-step approach:

1. Plain text LLM call produces a clean markdown document
2. Lightweight structured call extracts a small metadata summary (counts, scores, gaps)

**Impact:** Eliminates PSD generation failures. Better prose quality. Markdown is now the primary format — no JSON in the client-facing flow. The Machine (downstream Python system) will handle its own markdown-to-JSON parsing later.

**Files changed:** 9 | **Lines:** -1230, +839

### v1.2.0 — Stakeholder Alignment
Driven by a stakeholder memo requesting premium pause/resume, energy pacing, and friendlier tone. Three features shipped:

1. **Pause & Get Notes** — new button generates an LLM-powered homework memo (what's covered, what's needed, action items)
2. **Gap tracker** — PSD metadata now includes a `gaps[]` array. Orange banner in UI. Machine can consume downstream.
3. **"Red Team Challenge" renamed to "Idea Strengthening"** — prompt reworded from adversarial to collaborative

Also adopted the remote's friendlier "App Idea Coach" conversation prompt (5 casual steps, no jargon, no phase numbers visible to users).

**Files changed:** 6 | **Lines:** +100

### v1.3.0 — UI Redesign
Complete frontend rewrite:

- **Dark theme removed** — warm light palette (Linear-inspired). White surfaces, soft shadows, warm grays, indigo accent.
- **3-column layout** — left sidebar (phase stepper with clickable history), center chat, right preview panel (progressive PSD that builds as phases complete)
- **Phase navigation** — click completed phases to review that conversation. "Back to current" pill to return.
- **Responsive** — sidebar collapses to icons at 1024px, horizontal strip at 768px
- **survey.html** and **one-pager generator** also updated to light theme
- **Zero dark theme colors remaining** in the entire codebase (verified via grep)

**Files changed:** 3 | **Lines:** -537, +559

---

## Resolved Issues (from prior audit)

| Issue | Resolution |
|-------|-----------|
| PSD generation Zod validation failures | Fixed — markdown output, no JSON schema |
| Dark/intimidating UI | Fixed — warm light theme, 3-column layout |
| Service not running | Fixed — live on Railway since v1.0.0 |
| No pause/resume experience | Fixed — Pause & Get Notes with homework memo |
| Session data lost on restart | Fixed in v1.0.0 — SQLite persistence |
| 13-check scoring too limited | Fixed in v1.0.0 (expanded to 22, now 14 markdown-based) |

---

## Open Issues (prioritized)

### Critical
| Issue | Risk | Notes |
|-------|------|-------|
| No admin auth on token endpoints | Anyone can create/list/delete tokens | `POST/GET/DELETE /api/admin/tokens` wide open |
| Token validation bypassed | Non-existent tokens accepted as valid | Session isolation is cosmetic |

### High
| Issue | Risk | Notes |
|-------|------|-------|
| No rate limiting | Single client can exhaust API budget | `express-rate-limit` not installed |
| Conversation history unbounded | Token costs grow linearly per session | No cap during regular chat calls |

### Medium
| Issue | Risk | Notes |
|-------|------|-------|
| CORS wide open | Any domain can call API | `app.use(cors())` with no config |
| No concurrent session protection | Parallel writes can overwrite | No optimistic locking |
| 50MB body size limit | DoS vector | Attachment validator runs after body parsing |
| DB cleanup memory-only | Stale sessions accumulate in SQLite | `cleanupExpired()` only evicts from Map |

---

## Test Summary

| Suite | Tests | Status |
|-------|-------|--------|
| psd-scoring.test.ts | 10 | Pass |
| session-manager.test.ts | 8 | Pass |
| middleware.test.ts | 4 | Pass |
| api-routes.test.ts | 12 | Pass |
| **Total** | **34** | **All passing** |

---

## Metrics

| Metric | Value |
|--------|-------|
| Releases today | 3 (v1.1.0, v1.2.0, v1.3.0) |
| Files changed | 18 (across all commits) |
| Tests | 34 passing, 0 failing |
| Dark theme colors remaining | 0 |
| PSD scoring checks | 14 (section + substance + meta) |
| Export formats | 4 (Markdown, JSON meta, Replit prompt, One-pager HTML) |

---

## Recommendation

The product is in strong shape for demos and early client use. The UI redesign makes it approachable for non-technical users. The critical security items (admin auth, rate limiting) should be addressed before any public-facing deployment beyond invite-only access.

**Next priorities:**
1. Admin auth middleware (30 min)
2. Rate limiting (15 min)
3. CORS lockdown (5 min)
4. Conversation history cap (1 hour)
