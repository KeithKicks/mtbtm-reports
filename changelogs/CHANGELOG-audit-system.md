# Audit & Self-Improvement Subsystem — Changelog

The audit subsystem is the self-improvement engine inside MTBTM. It watches how agents perform, detects bottlenecks, refines failing specs, and promotes agents through maturity levels -- all automatically as part of the daily operating loop.

---

## [1.0.0] — 2026-03-02 (Phase 04 + 05 combined release)

### Agents

**QualityAuditorAgent** (`agents/quality_auditor.py`)
- Evaluates agent runs against quality rubrics
- Per-dimension scoring with configurable weights and weighted average rollup
- Pass/fail determination against rubric threshold
- Batch mode: audit last N runs for any agent
- Mock mode varies scores by run status for testing
- Stores results in `quality_audits` table

**BottleneckDetectorAgent** (`agents/bottleneck_detector.py`)
- Analyzes human intervention patterns over configurable window (default 7 days)
- Groups interventions by agent name and top category
- Returns empty report with explanatory note when < 3 interventions (avoids noisy signal)
- LLM-powered recommendations for which specs to refine
- Mock mode with deterministic output for testing

**SpecRefinerAgent** (`agents/spec_refiner.py`)
- Takes a failing spec + intervention evidence, produces a NEW IPOSpec (original preserved)
- Frames all changes as testable hypotheses (not just edits)
- Versions prompt files with timestamp suffixes to maintain history
- Builds `SpecRefinement` records with failure pattern summaries
- Mock mode generates plausible refinement data

**SystemAdvisorAgent** (`agents/system_advisor.py`)
- Synthesizes all system data into a daily briefing
- Enforced editorial constraints: `top_action` max 2 sentences, `working_well` max 1 sentence
- Gathers data from: bottleneck reports, maturity levels, costs, interventions, open decisions, gaps, unbuilt specs
- History method returns last N briefings for trend analysis

### Quality Rubrics (`agents/rubrics.py`)

Four default rubrics with dimension weights and pass thresholds:

| Rubric | Dimensions (weight) | Threshold |
|--------|---------------------|-----------|
| Scout | source_diversity (0.3), url_verification_rate (0.4), relevance_quality (0.3) | 0.70 |
| Reverse Engineer | granularity_compliance (0.4), completeness (0.4), scope_correctness (0.2) | 0.75 |
| Synthesizer | checklist_compliance (0.5), deduplication (0.3), coverage (0.2) | 0.70 |
| Extractor | verification_pass_rate (0.5), session_completion_rate (0.3), spec_buildability (0.2) | 0.75 |

Idempotent seeding via `seed_default_rubrics(db)`. Versioned (`v1`) for future rubric evolution.

### Agent Maturity System (`agents/maturity.py`)

Three-tier promotion ladder:

| Level | Requirements |
|-------|-------------|
| Level 1 | Default for all new agents |
| Level 2 | 20+ runs, ≤20% intervention rate, at least 1 audit score ≥ 0.75 |
| Level 3 | 100+ runs, ≤5% intervention rate, 3 consecutive passing audits, no refinements in 30 days |

Maturity updates run automatically in the daily operating loop.

### Daily Operating Loop (`core/scheduler.py`)

11-step automated cycle:

| Step | Action |
|------|--------|
| 0 | Database backup (SQLite WAL, 7-day rolling retention) |
| 1 | Budget check (halts if over monthly budget) |
| 2 | WIP limit check (skips builds if max agents reached) |
| 3 | Bottleneck detection (7-day intervention window) |
| 4 | Auto-refine high-intervention agents (rate > 0.3) |
| 4.5 | Auto-refine failed validations (split/fix draft specs) |
| 4.75 | Efficiency audit + auto-remediation + weekly digest (Mondays) |
| 5 | Audit recent unaudited runs (last 24h) |
| 6 | Update agent maturity levels |
| 7 | Record metrics (golden signals snapshot) |
| 8 | Check SLOs (error rate, latency, cost) |
| 9 | Generate daily briefing with metrics enrichment |
| 10 | Auto-update manifest current_state |
| 11 | Auto-update changelog |

Supports `--dry-run` mode. Budget halt produces a special briefing explaining why the cycle stopped.

### Supporting Agents (added in Phase 05, v1.2.0)

- **EfficiencyAuditorAgent** — Scans for operational inefficiencies across agents
- **AutoRemediatorAgent** — Auto-fixes flagged issues or escalates for human review
- **ValidationRefinerAgent** — Auto-refines specs that fail validation checks
- **WeeklyDigestGenerator** — Produces markdown summary reports (runs on Mondays in step 4.75)

### Database Tables

| Table | Purpose |
|-------|---------|
| `quality_rubrics` | Rubric definitions with version tracking |
| `quality_audits` | Per-run audit scores and diagnostic details |
| `bottleneck_reports` | Intervention pattern analysis results |
| `spec_refinements` | Hypothesis-driven spec improvements |
| `daily_briefings` | System advisor outputs |
| `metrics_snapshots` | Golden signals time series |
| `slo_definitions` | SLO thresholds and severity levels |
| `slo_violations` | SLO breach records |
| `human_interventions` | Manual intervention log (feeds bottleneck detector) |

### CLI Commands

```
python run.py audit --run-id <id>           # audit a specific agent run
python run.py audit --agent <name> --last 20 # audit last N runs
python run.py bottlenecks --days 7           # detect bottlenecks
python run.py refine --spec-id <id>          # refine a failing spec
python run.py briefing                       # generate daily briefing
python run.py daily                          # run full daily operating loop
python run.py daily --dry-run                # dry run (no writes)
python run.py log-intervention               # log human intervention
python run.py metrics                        # golden signals report
python run.py slos                           # SLO status + error budgets
```

---

## Current Status (as of 2026-03-27)

| Component | State |
|-----------|-------|
| Quality rubrics seeded | No (0 rubrics in DB) |
| Quality audits run | 0 |
| Bottleneck reports | 2 |
| Daily briefings | 2 (2026-03-02, 2026-03-27) |
| Spec refinements | 403 |
| Human interventions | 7 |
| Metrics snapshots | 0 |
| SLO violations | 0 |
| Agents eligible for Level 2+ | spec_validator (692 runs), builder (667 runs) — blocked on audits |

> **Key gap:** The self-improvement flywheel is fully built but not yet spinning. Rubrics need to be seeded, audits need to run, and the daily loop needs to be scheduled. Once activated, spec_validator and builder can promote to Level 3 almost immediately.
