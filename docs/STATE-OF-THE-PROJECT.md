# State of the Project -- 2026-07-06

This file exists so that Codex can resume this project in any session without losing context. Read this first.

## Backstory

The user worked with ChatGPT to explore A-share long-term investment indicators and AI Agent architecture. ChatGPT produced a comprehensive conversation (Long-term investment chat with Chatgpt.md). Codex then synthesized ChatGPT output with its own A-share domain knowledge into COMPREHENSIVE_INVESTMENT_FRAMEWORK.md.

The user also has a reference project: ai-investment-mentor (16 phases, 5 stages) which provides proven patterns for Agent architecture, Pipeline design, MemoryRepository, and document standards.

## Where We Are Now

Stage 0 (Architecture and Philosophy) -- Phase 0 in progress.

- COMPREHENSIVE_INVESTMENT_FRAMEWORK.md: Complete. ~110 indicators across 12 dimensions, 3-layer funnel, 100-point scoring model.
- AGENTS.md: Complete. SDD workflow defined.
- PROJECT_RULES.md: Complete. All templates defined including SDD Spec.
- ROADMAP.md: Complete. 16 phases across 6 stages.
- 03-agent-architecture.md: Being written (this session).
- ADRs 001-004: Being written (this session).

## Key Decisions Made

1. **SDD (Specification Driven Development).** Every Phase produces a spec before code. The spec defines the contract. Code implements the contract. Tests verify the contract.
2. **Advisor-centric architecture.** One Chief Investment Agent orchestrates 7 specialist Agents. Specialists never call each other.
3. **Event-driven pipeline.** Not daily cron. Triggered by earnings releases or manual request.
4. **4 memory types.** Funnel, Company, Watchlist, Decision.
5. **3-layer funnel.** 5300 stocks -> 500 screened -> 100 tracked -> 20 deep-researched.

## Inherited from ai-investment-mentor

- 5-layer architecture (UI -> Agent -> Tool -> Component -> Data)
- Agents vs Components hard boundary
- Advisor-centric (hub-and-spoke) topology
- Pipeline DAG executor (asyncio)
- Degrade-gracefully error model
- Document templates (Agent, Component, ADR, TASK)
- SQLite MemoryRepository pattern

## What We Added

- SDD spec template in PROJECT_RULES.md
- A-share-specific indicators (扣非/归母, 收现比, 商誉, 大股东质押, PE/PB分位数)
- 4th memory type (Funnel Memory)
- Event-driven pipeline (not daily cron)
- Multi-preset scoring (Standard / Dividend / Growth)

## Current Phase

Phase 0 -- COMPLETE. All ADRs written. Agent architecture documented.
Next session: begin Phase 1 -- formal ADR review and approval.

## Session Summary (2026-07-06)

Completed this session:
- ROADMAP.md: 16 phases across 6 stages, SDD-enforced (every phase has a spec)
- AGENTS.md: SDD workflow defined, inherited patterns from ai-investment-mentor
- PROJECT_RULES.md: All templates (Agent, Component, ADR, Spec, TASK) with SDD Spec Template
- docs/architecture/03-agent-architecture.md: Complete Agent design (14KB+), including output contract with full JSON schema for research dossiers
- adr/ADR-001 through ADR-004: All four architecture decisions documented
- COMPREHENSIVE_INVESTMENT_FRAMEWORK.md: Domain reference (~110 indicators, 12 dimensions, 3-layer funnel)

Key decisions locked:
- Technology stack: Python 3.11+, SQLite, DeepSeek API (openai SDK), pure asyncio Pipeline (no LangChain, LangGraph deferred), AkShare, Streamlit dashboard (Phase 16), pytest
- Agent architecture: 10 Agents (1 deterministic Screener + 7 Analysis + 2 Context), Advisor-centric hub-and-spoke topology
- Output contract: No Naked Recommendations -- every stock carries complete analysis_data package
- Memory: 4 types (Funnel, Company, Watchlist, Decision)
- Pipeline: Event-driven (earnings release trigger), not daily cron

## Next Session

Phase 1 -- formal ADR review and approval. Once ADRs pass, write specs/phase-3-project-bootstrap.md (first SDD spec) and begin Stage 1. All design artifacts delivered and pushed to GitHub.
README.md added. Begin Phase 1 -- formal ADR review and approval. Once ADRs are approved, write specs/phase-3-project-bootstrap.md (first SDD spec) and begin Stage 1 (Data Foundation). (remaining: 01-product-vision.md, 02-system-overview.md), then begin Phase 1 formal ADR review and approval.

## Implementation Language

Python 3.11+. SQLite for persistence. AkShare for A-share data. DeepSeek API (OpenAI-compatible) for LLM.

## Language Convention

All project documentation in English. Domain-specific Chinese terms (stock names, indicator names like 扣非净利润) may appear in data and variable names.
