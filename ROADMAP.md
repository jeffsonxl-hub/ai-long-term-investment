<!--
  ============================================================================
  Roadmap: AI Long-Term Investment Analysis System -- SDD Edition

  16 phases across 6 stages. Specification Driven Development:
  every Phase produces a spec in specs/ BEFORE any code is written.

  Reference project: ai-investment-mentor (16 phases, 5 stages)
  ============================================================================
-->

# Roadmap: AI Long-Term Investment Analysis System (SDD)

16 phases across 6 stages. Each phase produces a spec first, then code.
Mark a phase complete only when its spec is approved AND its acceptance
criteria pass.

---

## SDD Rule (Project Law)

`
SPEC -> REVIEW -> TASK -> CODE -> VERIFY
`

| Artifact | Location | Purpose |
|----------|----------|---------|
| Spec | specs/phase-N-name.md | Binding contract: interfaces, behavior, acceptance criteria |
| TASK | tasks/TASK-NNN-name.md | Implementation instructions for Codex |

No .py file is created until the corresponding spec is reviewed and approved.

---

## Stage 0 -- Architecture and Philosophy

No code. Pure design. The cost of changing decisions here is near zero.

| Phase | Status | Core Question | Spec | Output |
|-------|--------|---------------|------|--------|
| Phase 0 | In Progress | Why build this and who is it for? | N/A (design doc) | docs/architecture/01-product-vision.md |
| Phase 1 | In Progress | Is this one Agent doing everything, or multiple Agents with a coordinator? | N/A (ADRs) | ADR-001 + ADR-002 |
| Phase 2 | In Progress | What does the system need to remember, and when does it run? | N/A (ADRs) | ADR-003 + ADR-004 |

**Stage 0 deliverable**: 4 ADRs + product vision + agent architecture doc.
All architectural decisions locked before any code is written.

---

## Stage 1 -- Data Foundation

No analysis yet. The question: can we reliably get A-share data?

| Phase | Status | Core Question | Spec | Output |
|-------|--------|---------------|------|--------|
| Phase 3 | Planned | What is our Python project structure? | specs/phase-3-project-bootstrap.md | Project skeleton + requirements.txt + pytest + config loader |
| Phase 4 | Planned | Can we fetch financial statements from AkShare? | specs/phase-4-data-client.md | Data client module (3 statements for one stock) |
| Phase 5 | Planned | Can we cache, validate, and store this data? | specs/phase-5-data-cache.md | MemoryRepository (SQLite) + data quality + cache |

**Stage 1 deliverable**: run fetch 600519 -> validated 5-year financial data in SQLite.

---

## Stage 2 -- Indicator Engine

No scoring or Agent reasoning yet. Can we compute every indicator?

| Phase | Status | Core Question | Spec | Output |
|-------|--------|---------------|------|--------|
| Phase 6 | Planned | Can we compute profitability, growth, and efficiency indicators? | specs/phase-6-core-indicators.md | ~30 indicator calculators |
| Phase 7 | Planned | Can we compute cash flow, financial health, valuation, and A-share-specific indicators? | specs/phase-7-advanced-indicators.md | ~50 indicator calculators |

**Stage 2 deliverable**: compute_all_indicators("600519") returns dict with values, 5yr trends, and industry comparisons.

---

## Stage 3 -- Scoring and Screening

Given computed indicators, can we score and filter stocks?

| Phase | Status | Core Question | Spec | Output |
|-------|--------|---------------|------|--------|
| Phase 8 | Planned | Can we implement the 100-point scoring model? | specs/phase-8-scoring-engine.md | Scoring engine + penalty engine + score storage |
| Phase 9 | Planned | Can we run the three-layer funnel end-to-end? | specs/phase-9-funnel-pipeline.md | Funnel pipeline (coarse -> fine -> decision) |

**Stage 3 deliverable**: screen --pool sh50 returns ranked list with scores, funnel layers, and penalty flags.

---

## Stage 4 -- Agent Layer

Can LLM-powered Agents produce qualitative analysis?

| Phase | Status | Core Question | Spec | Output |
|-------|--------|---------------|------|--------|
| Phase 10 | Planned | Can an Agent produce structured quality analysis? | specs/phase-10-quality-agents.md | Quality Agent + Governance Agent |
| Phase 11 | Planned | Can Agents cover industry, risk, and macro context? | specs/phase-11-context-agents.md | Industry Agent + Risk Agent + Macro Agent |

**Stage 4 deliverable**: For one stock: (1) indicator report, (2) composite score with breakdown, (3) qualitative Agent analysis explaining the scores.

---

## Stage 5 -- Portfolio and Delivery

The system does analysis. Now make it usable.

| Phase | Status | Core Question | Spec | Output |
|-------|--------|---------------|------|--------|
| Phase 12 | Planned | Can we produce a readable investment research report? | specs/phase-12-chief-agent.md | Chief Investment Agent |
| Phase 13 | Planned | Can the system manage a portfolio? | specs/phase-13-portfolio-agent.md | Portfolio Manager Agent |

**Stage 5 deliverable**: End-to-end: select 5-10 stocks -> full analysis each -> portfolio summary.

---

## Stage 6 -- Operations and Polish

The system works. Make it reliable and self-improving.

| Phase | Status | Core Question | Spec | Output |
|-------|--------|---------------|------|--------|
| Phase 14 | Planned | Can the system auto-detect earnings releases? | specs/phase-14-event-watcher.md | Event watcher |
| Phase 15 | Planned | Can the system learn from its own recommendations? | specs/phase-15-feedback-loop.md | Feedback loop |
| Phase 16 | Planned | Can we surface everything in a clean dashboard? | specs/phase-16-dashboard.md | Web dashboard |

**Stage 6 deliverable**: Auto-triggered re-analysis, outcome tracking, visual dashboard.

---

## Phase Dependency Map

`
Stage 0:  [0 -> 1 -> 2]          (sequential, all design)
Stage 1:  [3 -> 4 -> 5]          (sequential)
Stage 2:  [6 -> 7]               (sequential within stage)
Stage 3:  [8 -> 9]               (sequential)
Stage 4:  [10, 11]               (parallel -- different Agents)
Stage 5:  [12 -> 13]             (sequential)
Stage 6:  [14 -> 15 -> 16]       (sequential)
`
