# MTBTM Reports

Weekly audits, changelogs, and stakeholder reports for **The Machine That Builds Itself** and its subsystems.

---

## Build vs. Buy: What This Portfolio Would Have Cost

Everything below was built by **one person + AI** in **15 days** (March 16-30, 2026).

Here's what the same scope would have cost a company hiring a traditional dev team just a few years ago.

### What Was Built

| System | What It Is | Scale |
|--------|-----------|-------|
| **MTBTM Core Machine** | Self-improving AI agent platform with 24 agents, 30+ database tables, full CLI, REST API, quality auditor, bottleneck detector, spec refiner, maturity system, cost tracking, SLO monitoring | ~15,000 lines, 547 tests |
| **Discovery Agent** | 7-phase conversational AI that turns ideas into validated product specs with viability scoring, competitive analysis, personas, red team challenge, 22-check scoring rubric, multi-format export, full UI, Railway deploy | ~4,500 lines, 35 tests |
| **Night Receptionist** | Complete multi-tenant SaaS -- AI receptionist with Twilio voice/SMS, Google Calendar, embeddable chat widget, React dashboard, agency admin portal, CRM, analytics, team RBAC, notifications | 12 releases, full product |
| **Audit & Ops Tooling** | Automated weekly audits, changelog management, GitHub-hosted stakeholder reports | Supporting infrastructure |

### Traditional Team Cost Estimate (~2020 pricing)

| Work Area | Estimated Hours | Notes |
|-----------|:-:|-------|
| MTBTM architecture + core framework | 500 | DB layer, permissions, models, scheduler, cost tracker, cache |
| 12 production agents | 600 | Orchestrator, validators, builders, extractors, auditors |
| Self-improvement loop | 300 | Quality auditor, bottleneck detector, spec refiner, maturity -- novel R&D in 2020 |
| Observability layer | 200 | Metrics, SLOs, structured logging, FTS5 fact store |
| Pipeline + codegen + deploy | 300 | State machine, PSD decomposer, template codegen, Docker provider |
| CLI + API + 547 tests | 500 | 20+ CLI commands, FastAPI REST API, full test suite |
| Discovery Agent (full build) | 800 | 7-phase conversational AI, PSD schema with viability scoring + personas + competitive analysis + roadmap, 22-check scoring, session persistence, token auth, multi-format export, survey form, UI, deploy |
| Night Receptionist (full SaaS) | 1,700 | AI engine, Twilio, Calendar, React dashboard, widget, agency portal, CRM, analytics, RBAC |
| Reports + ops tooling | 50 | Audit automation, changelog management |
| **Total** | **4,950** | |

### The Numbers

| | Traditional Team | Solo + AI |
|---|:-:|:-:|
| **People** | 3-4 senior engineers | 1 |
| **Timeline** | 7-8 months | 15 days |
| **Estimated cost** | **$1M - $1.4M** | A fraction of that |

> **How the traditional estimate breaks down:**
> - In-house team (3-4 engineers, 7-8 months fully loaded): $500K - $750K
> - Mid-tier consulting firm at ~$225/hr: $900K - $1.2M
> - AI specialty shop / top-tier agency at ~$300/hr: $1.3M - $1.7M
>
> **Why it skews high:** In 2020, self-improving agent orchestration was cutting-edge R&D. The talent pool for "build me a platform where AI agents audit and improve each other" was extremely small. Teams would have burned weeks on failed approaches before writing production code. Budget 30-40% overhead for iteration and dead ends.

### What the Speed Difference Means

The 15-day build vs. 7-8 month traditional timeline isn't just faster -- it changes the economics entirely:

- **Iteration speed**: 12 releases of Night Receptionist, Discovery Agent from v0.1 to v1.0 in 15 days. A traditional team ships maybe 2-3 in that window.
- **No coordination overhead**: No standups, no sprint planning, no PR review bottlenecks, no knowledge silos between team members.
- **Built-in quality**: 580+ tests written alongside the code, not bolted on after. Zero test failures.
- **Full-stack range**: Backend (Python + Node), frontend (React), infrastructure (Docker, Railway), AI/ML (Claude, OpenAI), telephony (Twilio), calendar (Google), database (SQLite + PostgreSQL) -- one person covered it all.

---

## Audits

| Date | System | Status | Link |
|------|--------|--------|------|
| 2026-03-27 | MTBTM (Core Machine) | Stable / Idle | [View Report](audits/audit-2026-03-27-mtbtm.md) |
| 2026-03-30 | Discovery Agent v1.0.0 | Live on Railway | [View Report](audits/audit-2026-03-27-discovery-agent.md) |

## Changelogs

| System | Link |
|--------|------|
| MTBTM (Core Machine) | [View Changelog](changelogs/CHANGELOG-mtbtm.md) |
| Discovery Agent | [View Changelog](changelogs/CHANGELOG-discovery-agent.md) |
| Audit & Self-Improvement Subsystem | [View Changelog](changelogs/CHANGELOG-audit-system.md) |
| Night Receptionist | [View Changelog](changelogs/CHANGELOG-night-receptionist.md) |

---

### Quick Stats (as of 2026-03-30)

| Metric | MTBTM | Discovery Agent | Night Receptionist |
|--------|-------|-----------------|-------------------|
| Tests | 547 passing | 35 (4 suites) | -- |
| Agent Runs | 1,377 (0% failure) | Live on Railway | -- |
| Specs | 821 (766 built) | N/A | -- |
| Cost per run | ~$50/MVP, ~$250/full build | ~$3/session | -- |
| SLOs | All green | No SLOs defined | -- |
| Version | Phase 06 complete | v1.0.0 | v0.12.0 |
