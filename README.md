# AI Long-Term Investment Analysis System

An AI-powered research platform for A-share long-term value investing. Not a trading bot -- a research assistant that analyzes ~110 indicators across 12 dimensions, runs 10 specialized AI Agents, and produces research dossiers you can verify and act on.

> **Principle**: AI narrows 5,300 A-shares to ~20 deep-researched candidates. You make the final decision.

---

## What It Does

`
5300+ A-shares
    |
    v  Screener Agent (hard thresholds: ROE > 15%, OCF/NI > 0.7, ...)
    |
  ~500 screened
    |
    v  7 Analysis Agents + 2 Context Agents (parallel LLM analysis)
    |
  ~100 tracked  -->  ~20 deep-researched
    |
    v  Chief Investment Agent (synthesis + research dossier)
    |
  Research Dossier
  (composite score, dimension breakdown, evidence chain, investment thesis)
`

Every stock that reaches the deep-research stage gets a complete dossier -- not a one-line "Buy" recommendation, but a full analysis package with dimension scores, key indicators, Agent narratives, quarter-over-quarter comparison, and a traceable evidence chain.

## Architecture

| Layer | Role |
|-------|------|
| **Agent Layer** | 10 Agents (1 deterministic Screener + 7 Analysis + 2 Context) coordinated by a Chief Investment Agent in hub-and-spoke topology |
| **Tool Layer** | Indicator Engine (110+ deterministic computations), Data Fetcher (AkShare), LLM Client (DeepSeek) |
| **Component Layer** | MemoryRepository (SQLite, 4 memory types), Pipeline (asyncio DAG executor), ConfigLoader, Logger |
| **Data Layer** | AkShare (primary),  (fallback), SQLite, News APIs |

### 10 Agents

| Agent | Role |
|-------|------|
| **Screener** | Hard-threshold filtering -- deterministic, no LLM |
| **Quality** | Profitability + efficiency: ROE, ROIC, margins, DuPont |
| **Growth** | Historical + projected growth: revenue/profit/EPS CAGR |
| **Cash Flow** | Profit quality verification: OCF/NI, cash collection ratio |
| **Valuation** | Multi-model fair value: DCF, PE/PB percentiles, margin of safety |
| **Financial Health** | Solvency + A-share red flags: goodwill, pledge ratio, interest coverage |
| **Governance** | Management quality: insider trading, turnover, institutional holdings |
| **Risk** | Devil's advocate -- negative-only scoring across all dimensions |
| **Industry** | Competitive dynamics: Porter's Five Forces, policy direction |
| **Macro** | Macroeconomic environment: GDP, CPI, M2, rates, RMB |
| **Chief Investment** | Orchestrator + synthesizer: composite score, cross-Agent analysis, dossier generation |

## Key Design Decisions

All documented as Architecture Decision Records:

| ADR | Decision |
|-----|----------|
| [ADR-001](adr/ADR-001-agent-architecture.md) | Hub-and-spoke topology -- Chief Investment Agent orchestrates 9 specialists |
| [ADR-002](adr/ADR-002-agent-boundaries.md) | Hard boundaries -- Chief never accesses data directly, specialists never call each other, all memory through MemoryRepository |
| [ADR-003](adr/ADR-003-memory-strategy.md) | 4 memory types (Funnel, Company, Watchlist, Decision) on SQLite |
| [ADR-004](adr/ADR-004-event-driven-pipeline.md) | Event-driven (earnings release trigger), not daily cron |

## Technology Stack

| Component | Choice |
|-----------|--------|
| Language | Python 3.11+ |
| Database | SQLite (zero-config, embedded) |
| LLM | DeepSeek API (OpenAI-compatible, via openai SDK -- no LangChain) |
| Orchestration | Pure asyncio Pipeline (custom DAG executor, LangGraph deferred) |
| Data | AkShare (primary),  (fallback) |
| Dashboard | Streamlit (Phase 16) |
| Testing | pytest |

## Project Status

**Phase 0 -- Complete** (Architecture & Philosophy)

- [x] Product vision + anti-goals
- [x] System overview (5-layer architecture)
- [x] Agent architecture (10 Agents, full output contract)
- [x] 4 ADRs (topology, boundaries, memory, pipeline)
- [x] SDD workflow (spec-before-code enforced)
- [x] 16-phase roadmap across 6 stages

**Next: Phase 1-2 -- ADR Review & Approval**

## Documents

| Document | Purpose |
|----------|---------|
| [Product Vision](docs/architecture/01-product-vision.md) | Who this is for, what success looks like |
| [System Overview](docs/architecture/02-system-overview.md) | 5-layer architecture, data flow, invariants |
| [Agent Architecture](docs/architecture/03-agent-architecture.md) | 10-Agent design with output contract (JSON schema) |
| [Investment Framework](COMPREHENSIVE_INVESTMENT_FRAMEWORK.md) | ~110 indicators across 12 dimensions |
| [Roadmap](ROADMAP.md) | 16 phases, SDD-enforced |

## Development Workflow

This project uses **Specification Driven Development (SDD)**:

`
SPEC -> REVIEW -> TASK -> CODE -> VERIFY
`

No .py file is written until the corresponding specs/phase-N-name.md is reviewed and approved. The spec defines the binding contract (interfaces, behavior, acceptance criteria). Code implements the contract. Tests verify the contract.

## Getting Started (Coming in Phase 3)

`ash
# Phase 3+ (not yet implemented)
git clone https://github.com/jeffsonxl-hub/ai-long-term-investment.git
cd ai-long-term-investment
python -m venv .venv
.venv\Scripts\activate
pip install -r requirements.txt
cp .env.example .env   # add your DeepSeek API key
`

---

*Built with guidance from [ai-investment-mentor](https://github.com/jeffsonxl-hub/ai-investment-mentor) -- a reference project that established the Agent architecture patterns, Pipeline design, and SDD workflow inherited here.*
