# MTBTM v1 — Changelog

**The Machine That Builds Itself**

All notable changes to this project are documented in this file.
Format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

---

## [1.9.0] — 2026-03-27 · Forge Lite Hardening + Model Fixes

Fixes and improvements uncovered while running real PSDs through the full
pipeline end-to-end.

### Fixed
- **Forge Lite API auth**: Generated apps used `Authorization: Bearer` header
  which Anthropic rejects. Fixed `call_mtbtm_api()` template to use
  `x-api-key` header + `anthropic-version: 2023-06-01`.
- **Model ID resolution**: Generated apps defaulted to `claude-haiku-4-5`
  (invalid). Validated correct model ID (`claude-haiku-4-5-20251001`) against
  live API and updated env config.
- **PSD field mapping**: Discovery Agent PSDs use `meta.productName` but
  Forge Lite expected `meta.projectName`. Import step now handles both field
  names.
- **Forge Lite `pydantic[email]`**: `requirements.txt` missing `[email]` extra
  for `EmailStr` validation. Apps crashed on import without it.

### Changed
- **Forge Lite templates**: `docker-compose.yml` and `.env.example` templates
  updated with correct `MTBTM_PROXY_URL`, `MTBTM_API_KEY`, and `CLAUDE_MODEL`
  defaults.
- **Spec validation at scale**: Validated 148 specs in a single batch run.
  Validator correctly caught 14 over-scoped specs (bundling multiple
  transformations) — confirms Check 2 logic is working as designed.

### Added
- **Procfile** added to Forge Lite output for Railway/Heroku deployment
  compatibility.
- **`.gitignore`** added to Forge Lite output (excludes `__pycache__/`, `.env`,
  `.venv/`).

---

## 2026-03-23 — Daily Activity

- 52 new project(s) created
- 76 new IPO spec(s) drafted
- 6 deployment(s) launched
- 5 database migration(s) applied

## [1.8.0] — 2026-03-23 · API Marketplace + Auto-Changelog

### Added
- **API Proxy Gateway** (`proxy/`): Standalone FastAPI service at `:9200` that
  meters, rate-limits, and forwards requests to Anthropic + OpenAI using
  platform keys. Clients never see provider credentials.
- **Tenant management**: `python run.py tenant create/list/keys/usage/revenue`
- **Usage metering**: Per-request logging with cost tables for 12 models
- **Transparent pricing**: `GET /v1/pricing` endpoint
- **Auto-changelog**: Daily loop appends activity summary to `docs/CHANGELOG.md`.
  Stakeholders view at `/changelog` in the browser.
- **Public changelog page**: `GET /changelog` — dark-themed HTML rendering of
  the changelog, auto-updated daily. Bookmark and share with stakeholders.
- Templates migrated: generated apps now use `MTBTM_PROXY_URL` + `MTBTM_API_KEY`
  instead of embedding `ANTHROPIC_API_KEY` directly.
- 25 new marketplace tests, 547 total passing.

## [1.7.0] — 2026-03-23 · AAR Bug Fixes & Consistency

After-action review of the Engineering Excellence release uncovered real bugs
and consistency issues. This release fixes them.

### Fixed
- **Rate limiter memory leak**: The in-memory IP tracker (`_request_log`) grew
  unbounded when hit by many unique IPs. Now prunes stale keys every 60 seconds
  and caps tracked IPs at 10,000 with LRU eviction.
- **Alert webhook spam**: SLO violations and budget alerts re-fired on every
  daily cycle with no deduplication. Alerts now check the `alert_log` table and
  suppress re-sends of the same `(alert_type, title)` within 60 minutes.
- **Silent backup failures**: `backup_database()` returned empty string on error,
  indistinguishable from "no DB found". Now returns a `BackupResult` dataclass
  with `status` / `path` / `error` fields. Failures fire a webhook alert via
  `alert_backup_failure()`.

### Changed
- **Standardized error handling**: All 51 top-level route exception handlers now
  use `safe_error_response(e, "context")` — which logs the error via Python's
  logging module AND returns consistent `{"ok": false, "error": "..."}`. No
  more silent `except Exception: pass` at the top level.
- **Broke circular import**: Extracted Pydantic models and shared helpers
  (`get_db`, `get_manifest`, `pf`) into `api_models.py`. Routes import from
  `api_models` instead of `api.py`. Eliminates the fragile `api ↔ routes`
  circular dependency.
- **Consolidated migration systems**: Legacy `_REQUIRED_MIGRATIONS` dict marked
  as deprecated with clear comments. `core/migrations.py` documented as the
  canonical system for all new schema changes. No-op baseline migration replaced
  with proper naming.

### Added
- `routes/__init__.py` — `safe_error_response()` shared error handler
- `api_models.py` — canonical home for Pydantic models + shared helpers
- `core/alerting.alert_backup_failure()` — backup failure alert
- `Retry-After` header on 429 rate-limit responses

### Test Results
- **522 tests**, all passing (0 regressions from 1.6.0)

---

## [1.6.0] — 2026-03-23 · Engineering Excellence (Phases A–C)

Major engineering upgrade bringing the project to world-class standards.
Three phases delivered back-to-back: Foundations, CI & Safety, Production Polish.

### Phase A: Foundations

#### Added
- **`pyproject.toml`**: Single config file for project metadata, ruff, mypy,
  and pytest. Replaces `requirements.txt` (loose) + `pytest.ini` (separate).
- **Ruff linter + formatter**: 498 lint issues auto-fixed across the codebase.
  Rules: pyflakes, pycodestyle, isort, pyupgrade, flake8-bugbear, flake8-simplify.
- **Mypy type checker**: Configured as informational (not blocking yet).
  218 pre-existing type issues identified for gradual cleanup.
- **Pre-commit hooks**: `.pre-commit-config.yaml` with ruff lint, ruff format,
  and mypy — runs automatically before every commit.
- **Makefile**: Standard targets — `make lint`, `make format`, `make typecheck`,
  `make test`, `make check` (CI gate), `make serve`, `make daily`, `make clean`.
- **`cli/` package**: Monolithic `run.py` (2,359 lines) split into 9 focused
  modules. `run.py` is now a 160-line dispatcher. Modules:
  - `cli/session.py` — session start/end
  - `cli/permissions.py` — permission profile management
  - `cli/phase1.py` — reference intelligence pipeline
  - `cli/gaps.py` — gap analysis + extraction
  - `cli/specs.py` — validate, build, refine, validate-refine
  - `cli/workflow.py` — workflow execution + trace viewer
  - `cli/ops.py` — daily loop, briefing, costs, metrics, SLOs, audit, facts
  - `cli/pipeline.py` — intake, forge-lite, codegen, deploy
  - `cli/improve.py` — efficiency audit + remediation

#### Changed
- Dependencies pinned to exact versions (was `>=`, now `==`)
- `datetime.utcnow()` replaced with `datetime.now(UTC)` (deprecation fix)
- All `timezone.utc` references upgraded to `UTC` alias (Python 3.11+)

### Phase B: CI & Safety

#### Added
- **GitHub Actions CI** (`.github/workflows/ci.yml`): Three parallel jobs —
  lint, typecheck (informational), test. Runs on push to `main` and all PRs.
- **CORS origin lock**: Was `allow_origins=["*"]` (any website could hit the
  API). Now locked to `localhost:8000` by default, configurable via
  `CORS_ORIGINS` env var.
- **API key authentication**: Optional middleware activated by setting `API_KEY`
  env var. All `/api/` routes require `X-API-Key` header when enabled. Health
  and intake endpoints are exempt. Returns 401 with JSON error body.
- **SQLite backup**: Automatic database backup as Step 0 of the daily operating
  loop. Uses SQLite's online backup API for consistency. Rolling 7-day retention
  with automatic pruning. Backups stored in `data/backups/`.
- 5 new tests for backup + API key middleware

#### Changed
- `.env.example` updated with `API_KEY`, `CORS_ORIGINS` config vars

### Phase C: Production Polish

#### Added
- **Route modules** (`routes/`): Monolithic `api.py` (1,874 lines) split into
  8 focused modules with `APIRouter`. `api.py` is now a 100-line app setup +
  router includes. Modules:
  - `routes/projects.py` — project CRUD (6 endpoints)
  - `routes/specs.py` — specs + artifacts (11 endpoints)
  - `routes/phase1.py` — phase 1 pipeline (9 endpoints)
  - `routes/operations.py` — briefings, costs, metrics, SLOs, facts,
    permissions, interventions, audits, workflow (20 endpoints)
  - `routes/pipeline.py` — pipeline, codegen, deploy, forge-lite (12 endpoints)
  - `routes/intake.py` — conversational intake (4 endpoints)
  - `routes/improvement.py` — efficiency dashboard (12 endpoints)
- **Rate limiting**: In-memory sliding-window rate limiter on all `/api/` paths.
  Default: 120 requests/minute/IP. Configurable via `RATE_LIMIT_PER_MINUTE`.
  Returns 429 with `Retry-After` header.
- **Webhook alerting** (`core/alerting.py`): Slack-compatible webhook alerts
  for SLO violations and budget breaches. Configure via `ALERT_WEBHOOK_URL` env
  var. Integrated into daily operating loop budget check and SLO check steps.
- **Deploy rollback**: `python run.py deploy --rollback <id>` tears down the
  current deployment and redeploys the previous codegen version. Marks current
  version as `rolled_back` in the database.
- **Numbered migration runner** (`core/migrations.py`): Version-tracked
  migrations with `schema_migrations` table. Each migration runs exactly once.
  Replaces the ad-hoc `PRAGMA table_info` column-add pattern for all new
  schema changes.
- `.env.example` updated with `ALERT_WEBHOOK_URL`, `RATE_LIMIT_PER_MINUTE`

### Test Results
- **522 tests**, all passing (up from 514)
- **Ruff lint**: 0 errors
- **Ruff format**: 90 files clean
- **Mypy**: runs (informational, 218 pre-existing issues)

---

## [1.5.0] — 2026-03-18 · IntakeAgent + Forge Lite

End-to-end product creation: describe an idea in plain language, get a live
deployed demo.

### Added
- **IntakeAgent** (`agents/intake.py`): Conversational front desk that guides
  users through a 5-step product intake process — Understand Idea, Target Users,
  MVP Features, Acceptance Criteria, Review & Approve. Modeled after the
  ExtractorAgent's Socratic pattern with history compression.
- **Forge Lite** (`agents/forge_lite.py`): Fast-lane MVP generator. Takes a PSD
  (Product Specification Document) and produces a complete single-file FastAPI
  app in one Claude call. Skips decomposition, validation, and artifact
  scaffolding for speed.
- **Full idea-to-deploy loop**: `intake → PSD → forge-lite → deploy`. A
  non-technical user describes their idea, the system interviews them, generates
  a PSD, creates a working app, deploys it with Docker, and returns a live URL.
- CLI: `python run.py intake` (interactive), `python run.py forge-lite --file <path>`
- API: `POST /api/intake/start`, `/message`, `/approve`, `POST /api/forge-lite`
- UI: `intake.html` — chat interface with PSD preview, approval flow, and
  "View Your Demo" button with live URL
- `intake_sessions` table for conversation persistence
- 26 new tests (15 intake + 11 forge-lite)

---

## [1.4.0] — 2026-03-12 · Code Generation + Deployment

From scaffolded specs to running code in Docker containers.

### Added
- **CodeGenAgent** (`agents/codegen.py`): Template-based code generation from
  scaffolded artifacts. Each IPO spec becomes a FastAPI app wrapping Claude.
  Rule-based type parsing from JSON examples with LLM fallback.
- **DeployAgent** (`agents/deployer.py`): Pluggable deployment providers via
  `DeployProvider` ABC. Ships with `DockerLocalProvider` (docker compose),
  `UvicornLocalProvider` (direct uvicorn), and `MockProvider` for tests.
  Provider selection via `DEPLOY_PROVIDER` env var.
- 8 code generation templates in `templates/codegen/` — app, Dockerfile,
  requirements.txt, test, docker-compose, gateway, .env.example, README
- Generated output: `generated/{project-slug}/{agent-slug}/` with
  project-level docker-compose and gateway reverse proxy
- `codegen_outputs` and `deployments` tables
- 5 pipeline API endpoints for codegen and deploy lifecycle
- UI: Generate Code, Deploy, Stop buttons; deploy status card with URL + logs
- CLI: `python run.py codegen --project <id>`, `python run.py deploy --project <id>`

---

## [1.3.0] — 2026-03-12 · Pipeline Handoff Layer

Connects the Discovery Agent to the build pipeline with a state machine.

### Added
- **Pipeline state machine**: 7 states with enforced transitions —
  `psd_approved → reviewing → confirmed → building → ready → delivered → archived`
- `confirmed → building` transition triggers automatic PSD decomposition into
  IPO specs via `PSDDecomposerAgent`
- **Discovery Agent polling**: `GET /api/pipeline/poll-discovery` imports PSDs
  from the external discovery service at `localhost:3001`
- 5 pipeline API endpoints + Pipeline UI panel with inbox, detail view, notes
- Project fields for pipeline tracking: `discovery_project_id`, `psd_json`,
  `pipeline_notes`, `deploy_url`, `deploy_status`

---

## [1.2.0] — 2026-03-02 · Improvement Layer

Automated efficiency auditing and self-healing.

### Added
- **EfficiencyAuditorAgent** (`agents/efficiency_auditor.py`): Scans the system
  for inefficiencies — expired caches, high intervention rates, stale specs,
  cost anomalies. Produces findings with severity levels.
- **AutoRemediatorAgent** (`agents/auto_remediator.py`): Automatically fixes
  auto-fixable audit findings (clear expired caches, etc.). Flags
  human-required fixes for review.
- **ValidationRefinerAgent** (`agents/validation_refiner.py`): Auto-refines
  specs that fail validation. Splits over-scoped specs or fixes specific check
  failures.
- **WeeklyDigestGenerator** (`core/weekly_digest.py`): Generates weekly markdown
  summaries of findings, remediations, and system health trends.
- `audit_findings`, `efficiency_reports`, `remediation_actions`,
  `weekly_digests` tables
- `improvement.html` — dashboard UI for findings, reports, and weekly digests
- CLI: `python run.py improve --audit`, `--remediate`, `--findings`, `--report`,
  `--weekly`, `--dashboard`

---

## [1.1.0] — 2026-03-02 · Phase 06: Observability & Semantic Upgrade

Production-grade observability and full-text knowledge search.

### Added
- **FactStore** (`core/fact_store.py`): FTS5 virtual table for semantic fact
  search with BM25 ranking. Automatic LIKE fallback. Content-sync triggers.
- **StructuredLogger** (`core/structured_logger.py`): JSON structured logging
  to stdout. Correlation IDs per daily cycle. Auto-captures agent log calls.
- **MetricsCollector** (`core/metrics.py`): 4 golden signals — latency
  (p50/p95), traffic/hour, error rate, saturation (cost % of budget).
- **SLOMonitor** (`core/slo.py`): 3 default SLOs (error rate 10%, latency 30s,
  cost 150%). Error budget tracking. Critical violations halt the daily cycle.
- **Health endpoint** (`GET /ready`): Returns ready/degraded/not_ready with SLO
  violation details.
- Console stderr separation for clean JSON stdout piping
- Tables: `metrics_snapshots`, `slo_definitions`, `slo_violations`,
  `semantic_facts_fts` (FTS5)

---

## [1.0.0] — 2026-03-02 · Phase 05: Integration & Automation

The machine runs itself daily.

### Added
- **Branching workflows**: OrchestratorAgent Stage 2 — `on_success`/`on_failure`
  routing, quality gates, `notify_owner` step, dry-run mode.
- **CostTracker** (`core/cost_tracker.py`): Budget enforcement at 80% warning.
  Daily/weekly/monthly cost tracking per agent.
- **ResponseCache** (`core/cache.py`): Deterministic SHA256 cache with
  configurable TTL. SQLite-backed.
- **Agent Maturity** (`agents/maturity.py`): Three-tier maturity system —
  Level 1 (default), Level 2 (20+ runs, ≤20% intervention, 1 audit ≥0.75),
  Level 3 (100+ runs, ≤5% intervention, 3 consecutive passing audits).
- **DailyOperatingLoop** (`core/scheduler.py`): Automated 9-step daily cycle —
  budget check → WIP check → bottleneck detect → auto-refine → audit → maturity
  update → metrics → SLO check → briefing. Dry-run mode. Budget halt. SLO halt.
- **Manifest auto-update**: Queries DB for system state and writes current
  status to the manifest automatically.
- `cron_daily.sh` for crontab scheduling
- Stress test framework for multi-domain pipeline validation

---

## [0.4.0] — 2026-03-02 · Phase 04: Self-Improvement Layer

The machine learns from its mistakes.

### Added
- **QualityAuditorAgent**: Evaluates agent runs against quality rubrics with
  per-dimension scoring and weighted averages.
- **BottleneckDetectorAgent**: Analyzes intervention patterns by agent and
  category. Recommends spec refinements.
- **SpecRefinerAgent**: Creates new IPO specs from failing specs + evidence.
  Preserves originals. Frames changes as testable hypotheses.
- **SystemAdvisorAgent**: Generates daily briefings with enforced format
  constraints.
- Default rubrics for scout, reverse_engineer, synthesizer, extractor
- Tables: `quality_rubrics`, `quality_audits`, `bottleneck_reports`,
  `spec_refinements`, `daily_briefings`

---

## [0.3.0] — 2026-02-27 · Phase 03: Construction Layer

Specs become real agent artifacts.

### Added
- **SpecValidatorAgent**: 5 binary validation checks (3 rule-based, 2
  LLM-based). Updates spec status on pass.
- **BuilderAgent**: Scaffolds 4 files per validated spec — prompt template,
  test cases, readme, mock response.
- **OrchestratorAgent Stage 1**: Linear workflow execution with fail-loud
  semantics. Cost tracking per step.
- Mock mode for all three agents

---

## [0.2.0] — 2026-02-27 · Phase 02: Gap Intelligence Layer

Find what's missing and define what to build.

### Added
- **GapForcerAgent**: Compares specs vs. master library, returns top 10 gaps by
  impact. Coverage gate warns if < 5 source systems.
- **ExtractorAgent**: Socratic interviewer — asks one question at a time until
  IPO spec is complete. Three verification checks. Session persistence. History
  compression after 10 turns.
- Multi-project architecture: 11 data tables with `project_id` column.
  `_fixup_legacy_projects()` migration. API filtering via `pf()` helper.

---

## [0.1.0] — 2026-02-26 · Phase 01: Foundation

The infrastructure everything else is built on.

### Added
- **DatabaseManager**: SQLite with WAL mode. Schema auto-creation. In-memory
  test mode.
- **Core data models**: `IPOSpec`, `AgentRun`, `AgentAction`,
  `HumanIntervention`, `ManifestSection`, `SemanticFact`, `AgentArtifact`,
  `Project` — all as Python dataclasses.
- **PermissionGate**: Least-privilege enforcement. `__default__` profile blocks
  5 universal danger patterns. `AgentPermissionProfile` registration.
- **ProjectManifest**: 8 mandatory sections (vision, current_state,
  component_registry, open_decisions, feedback_log, roadmap, decision_log,
  risk_register).
- **InterventionLogger**: Agent run, action, and intervention logging.
- **SessionManager**: Session start (briefing + stale fact review) and end
  (reconciliation with decision capture).
- **BaseAgentRunner**: Abstract base with trace ID generation, permission
  enforcement, retry logic (exponential backoff), and cost estimation.
- **Phase 1 pipeline**: ScoutAgent (find systems) → ReverseEngineerAgent
  (generate specs) → SynthesizerAgent (quality checks).
- FastAPI backend with ~80 endpoints, HTML UI, mock mode
- CLI entry point (`run.py`) with 30+ commands
- 4 Architecture Decision Records (ADRs)
- `.env`-based configuration

### Architectural Decisions
- Python 3.13, SQLite, Claude API, no ORM, no LangChain
  ([ADR-001](ADR-001-stack-decision.md))
- Manifest as SQLite table with opaque markdown sections
  ([ADR-002](ADR-002-manifest-design.md))
- Least privilege by default with permission gate
  ([ADR-003](ADR-003-access-controls.md))
- Structured logging + golden signals + SLOs
  ([ADR-004](ADR-004-observability-approach.md))

---

## Project Statistics

| Metric | Value |
|--------|-------|
| First commit | 2026-02-26 |
| Latest release | 2026-03-27 |
| Total tests | 522 |
| Python files | ~90 (excluding generated code) |
| Agent implementations | 25 |
| API endpoints | ~80 |
| Database tables | 30+ |
| ADRs | 4 |
| Lines of code | ~15,000 (core + agents + routes + cli) |

---

*Updated 2026-03-27. This changelog covers the complete project history from
initial foundation through production hardening and real-world pipeline testing.*
