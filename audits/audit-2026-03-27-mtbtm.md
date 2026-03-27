# MTBTM Weekly Audit Report
**Date:** 2026-03-27 | **Auditor:** Claude (automated) | **Period:** Week of 2026-03-21 to 2026-03-27

---

## Executive Summary

The Machine is stable and idle. All 547 tests pass, all 3 SLOs are green with 100% error budget, and total monthly spend is $0.012. The system has accumulated 1,377 successful agent runs with a **0% failure rate**. However, the Machine is not actively *doing* anything right now -- it's a well-built engine that's parked. The biggest opportunities are in activating the backlog and turning on the self-improvement loop.

---

## Golden Signals

| Signal | Value | Assessment |
|--------|-------|------------|
| Latency (p50/p95) | No data today | N/A -- no runs |
| Traffic | 0 runs today | Idle |
| Error Rate | 0% | Green |
| Saturation (cost % budget) | 0% | Green |
| Cache Hits (week) | 249 | Cache is working |

## SLO Status

| SLO | Threshold | Status | Budget Remaining |
|-----|-----------|--------|------------------|
| Agent Error Rate | < 10% over 7d | OK | 100% |
| Agent Latency p95 | < 30s over 7d | OK | 100% |
| Daily Cost | < 150% budget over 1d | OK | 100% |

## Cost

| Period | Spend |
|--------|-------|
| Today | $0.00 |
| This week | $0.00 |
| This month | $0.012 |
| Projected monthly | $0.00 |

---

## System Inventory

| Asset | Count |
|-------|-------|
| Agent runs (all-time) | 1,377 |
| IPO Specs | 821 (766 built, 32 validated, 19 draft, 4 in other states) |
| Agent artifacts | 766 (all maturity level 1) |
| Projects | 118 |
| Deployments | 18 (5 currently "running") |
| Codegen outputs | 68 |
| Intake sessions | 29 |
| Tests | 547 passing (0 failures) |
| Permission profiles | 15 (14 agents + __default__) |

---

## Findings

### CRITICAL: Quality Audit System Never Activated

The QualityAuditorAgent exists and is wired up, but:
- **0 quality audits** have ever been run
- **0 quality rubrics** are seeded in the database
- The `seed_default_rubrics()` function in `agents/rubrics.py` has never been called

**Why this matters:** The self-improvement loop (Phase 04) depends on quality audits to feed the BottleneckDetector and SpecRefiner. Without audits, agents can't mature past level 1, bottlenecks can't be detected, and specs can't be auto-refined. The entire self-improvement flywheel is disconnected.

**Suggestion:** Run `seed_default_rubrics()` once, then wire quality audits into the daily operating loop or run them manually on the 1,377 existing runs.

---

### HIGH: 32 Validated Specs Sitting Unbuilt

32 specs have passed validation but never been built into agent artifacts. This is the single largest actionable backlog item.

**Suggestion:** Run `python run.py build --all-validated` to scaffold all 32. This is low-risk since BuilderAgent has a 100% success rate across 667 prior runs.

---

### HIGH: No Metrics History

The `metrics_snapshots` table is empty. Golden signals are computed on-the-fly but never persisted. There's no way to trend system health over time.

**Suggestion:** The DailyOperatingLoop should snapshot metrics each run. Consider adding a metrics capture step or running `python run.py daily` on a schedule.

---

### HIGH: Semantic Fact Store is Empty

The FTS5-backed fact store has 0 entries despite infrastructure being fully built (virtual table, triggers, BM25 ranking). The system can't learn from its own operations.

**Suggestion:** Populate facts from the 2 existing daily briefings, 7 interventions, and 2 bottleneck reports. The FactStore was designed to accumulate institutional knowledge -- it needs to be seeded and fed.

---

### MEDIUM: DailyOperatingLoop Barely Used

Only 2 daily briefings exist (2026-03-02 and 2026-03-27). The 9-step daily cycle (budget check, WIP check, bottleneck detect, auto-refine, audit, maturity update, briefing, manifest update) hasn't been running regularly.

**Suggestion:** Set up the `cron_daily.sh` script or run `python run.py daily` manually at least a few times per week.

---

### MEDIUM: Duplicate Gap Entry

"Feedback Loop Agent" appears twice in the gap list (IDs `e0ec0964` and `4ca881f9`, both impact 7). This is data quality noise.

**Suggestion:** Delete one with `python run.py facts delete <id-prefix>` or directly via the gap management CLI.

---

### MEDIUM: machine.db is 0 Bytes on Disk

The SQLite database file exists but is empty. Data appears to be regenerated or lives in-memory at runtime. This means:
- No persistence across restarts unless the DB is rebuilt from migrations
- The 1,377 runs, 821 specs, etc. may be test/seed data rather than real operational history

**Suggestion:** Clarify whether the production database is at a different path or if the system is designed to be ephemeral.

---

### LOW: Git History is Minimal

Only 2 commits total. No branching, no tags, no activity this week. For a system with 547 tests and 15K lines of code, the git history is unusually sparse.

**Suggestion:** Consider more granular commits as changes are made, to support rollback and change tracking.

---

### LOW: 3 Open Architectural Decisions

OD-001 (agent communication pattern), OD-002, and OD-003 remain unresolved. Per the briefing, OD-001 is worth deciding before the backlog grows.

---

## Agent Maturity Assessment

| Agent | Runs | Failures | Maturity | Can Promote? |
|-------|------|----------|----------|-------------|
| spec_validator | 692 | 0 | Level 1 | Needs audits |
| builder | 667 | 0 | Level 1 | Needs audits |
| reverse_engineer | 4 | 0 | Level 1 | Too few runs |
| scout | 4 | 0 | Level 1 | Too few runs |
| auto_remediator | 3 | 0 | Level 1 | Too few runs |
| synthesizer | 3 | 0 | Level 1 | Too few runs |
| efficiency_auditor | 2 | 0 | Level 1 | Too few runs |
| extractor | 2 | 0 | Level 1 | Too few runs |

**Level 2 requires:** 20+ runs, <=20% intervention rate, 1 audit >= 0.75
**Level 3 requires:** 100+ runs, <=5% intervention rate, 3 consecutive passing audits, no refinements in 30 days

spec_validator and builder both meet the run-count and intervention-rate thresholds for Level 2 and Level 3 -- the only missing ingredient is quality audits.

---

## Top 5 Recommendations (Priority Order)

1. **Activate the self-improvement loop** -- Seed rubrics, run quality audits on existing runs, let maturity promotion happen. This is the core value proposition of the Machine.

2. **Build the 32 validated specs** -- `python run.py build --all-validated`. Immediate, low-risk progress.

3. **Schedule the daily operating loop** -- Even twice a week would generate metrics history, briefings, fact accumulation, and bottleneck detection.

4. **Seed the fact store** -- Extract learnings from the 7 interventions and 2 briefings. Give the system memory.

5. **Decide OD-001 (agent communication pattern)** -- This architectural decision gates how agents will coordinate as the system scales beyond linear workflows.

---

## Test Health

```
547 passed in 29.89s -- 0 failures, 0 errors, 0 skipped
```

Test count has grown from 454 (last documented) to 547. All green. Memory file should be updated to reflect 547.
