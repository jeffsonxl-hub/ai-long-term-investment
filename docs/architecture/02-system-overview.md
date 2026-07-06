# System Overview -- AI Long-Term Investment Analysis System

This document describes the high-level architecture: what layers exist,
how they communicate, and how a user request flows through the system.

## Architecture Layers

The system is organized into five layers. Each layer depends only on
the layer directly below it.

### User Interface

- **CLI (V1 primary)**: python -m src.cli analyze 600519 triggers a full
  analysis and outputs a research dossier as Markdown.
- **Streamlit Dashboard (Phase 16)**: Visual portfolio review with stock
  cards, dimension radar charts, score trend lines, and alert feed.

The UI layer contains no business logic. It receives structured JSON
from the Agent Layer and renders it.

### Agent Layer

10 Agents organized in a hub-and-spoke topology. See
03-agent-architecture.md for full design.

| Agent | LLM? | Role | Runs |
|-------|------|------|------|
| Screener | No | Hard-threshold filtering | Per batch |
| Quality | Yes | Profitability + efficiency | Per stock |
| Growth | Yes | Historical + projected growth | Per stock |
| Cash Flow | Yes | Profit quality verification | Per stock |
| Valuation | Yes | Multi-model fair value | Per stock |
| Financial Health | Yes | Solvency + balance sheet quality | Per stock |
| Governance | Yes | Management quality + alignment | Per stock |
| Risk | Yes | Negative-only risk detection | Per stock |
| Industry | Yes | Competitive dynamics + policy | Per batch |
| Macro | Yes | Macroeconomic environment | Per batch |
| Chief Investment | Yes | Orchestration + synthesis | Per stock |

Key rule: The Chief Investment Agent is the sole orchestrator.
Specialist Agents never call each other.

### Tool Layer

| Tool | Type | Purpose |
|------|------|---------|
| Indicator Engine | Component | Computes ~110 indicators from financial data. Deterministic, no LLM. The foundation every Agent depends on. |
| Data Fetcher | Tool | Fetches financial statements + market data from AkShare/Tushare. Timeout 30s, retry 1x. |
| LLM Client | Tool | Wraps DeepSeek API (OpenAI-compatible). Handles rate limiting, retry, token management. |
| News Scanner | Tool | Lightweight daily scan for Red Flag events on tracked stocks. |

### Component Layer

Infrastructure that persists across Agent invocations. Components are deterministic, have no LLM access.

| Component | Purpose | Inherited? |
|-----------|---------|------------|
| MemoryRepository | Read/write all four memory types via SQLite | Yes (adapted) |
| ConfigLoader | Load .env + environment, return frozen Config | Yes (direct) |
| Logger | Structured logging (JSON Lines) for auditing | Yes (direct) |
| Pipeline | DAG executor with asyncio, retry, severity model | Yes (direct) |

### Data Layer

| Source | Provides | Protocol |
|--------|----------|----------|
| AkShare | Financial statements, PE/PB history, sector data, northbound flow, earnings calendar | Python library |
| Tushare | Fallback for financial data | REST API |
| SQLite | Local persistence for all memory types | Local file |
| News APIs | Financial news for Risk Agent Watchdog scan (Phase 14) | REST API |
## Communication Patterns

1. **Agent to Tool**: Synchronous call-return. Tools do not know which Agent called them.
2. **Agent to Component**: All persistence through MemoryRepository. Agent never writes SQL.
3. **Agent to Agent**: NONE directly. Chief Investment Agent is the sole orchestrator.
   Specialist Agents never call each other.
4. **Tool to Data Layer**: Synchronous with 30s timeout and 1 automatic retry.
   Degradation over failure -- returns flag, not exception.

## Data Flow: Single Stock Analysis

`
User: python -m src.cli analyze 600519
  |
Step 1: Data Fetcher retrieves 5 years of financial statements
  |     (income statement, balance sheet, cash flow) from AkShare
  |     -> stored in SQLite cache
  |
Step 2: Indicator Engine computes ~110 indicators
  |     (deterministic, no LLM, ~100ms)
  |     -> IndicatorReport (dict)
  |
Step 3: Screener Agent verifies hard thresholds
  |     (ROE > 15%, OCF/NI > 0.7, Goodwill < 20%, Pledge < 30%)
  |     -> Pass/Fail + violation reasons
  |
Step 4: 7 Analysis Agents run IN PARALLEL
  |     Quality | Growth | CashFlow | Valuation | Financial | Governance | Risk
  |     (LLM calls for qualitative analysis + structured scoring)
  |     -> 7 AgentScore objects
  |
Step 5: 2 Context Agents provide industry + macro context
  |     (run once, cached for batch)
  |     -> IndustryScore + MacroScore
  |
Step 6: Chief Investment Agent synthesizes all outputs
  |     - Weighted composite score
  |     - Cross-Agent contradiction detection
  |     - Quarter-over-quarter comparison (vs Company Memory)
  |     - Narrative investment thesis
  |     - Research dossier (JSON + Markdown)
  |     -> saves to Company Memory + Decision Memory
  |
Step 7: Output delivered to user
        -> Markdown file: output/600519-2026-Q3.md
        -> Console: summary with link to full dossier
`

## Invariants (Must Always Hold)

1. **No layer-skipping.** An Agent cannot bypass Tools and directly access the Data Layer.
2. **Structured inter-Agent communication.** No Agent receives unstructured text from another.
3. **Chief Investment Agent is the sole orchestrator.** Specialist Agents respond; they do not self-trigger.
4. **Degradation over failure.** A failed layer returns a degradation flag, not an exception.
5. **Deterministic before LLM.** Indicator Engine and Screener run before any LLM is called.
6. **No naked recommendations.** Every output carries complete analysis data with evidence chains.

## Configuration and Environment

All configuration is externalized. No secrets, API keys, or environment-specific values are hardcoded.

- Environment variables: LLM_API_KEY, LLM_BASE_URL, LLM_MODEL, DB_PATH
- Configuration file (.env): Agent weights, scoring presets, data source preferences
- Code constants: Only definitionally constant values (e.g., SHANGHAI_COMPOSITE_INDEX_CODE)

Config is loaded once at startup into a frozen @dataclass(frozen=True) object. No runtime mutation.

## Technology Stack

| Component | Technology | Rationale |
|-----------|-----------|-----------|
| Language | Python 3.11+ | A-share data ecosystem, LLM SDK support |
| Database | SQLite via sqlite3 | Zero-config, embedded, single-user |
| LLM Provider | DeepSeek API (OpenAI-compatible) | Cost-effective, Chinese-literate |
| LLM SDK | openai (direct, no LangChain) | Simpler, no unnecessary abstraction |
| Orchestration | asyncio Pipeline (custom DAG) | Simple fan-out/fan-in, no loops yet |
| Data | AkShare (primary), Tushare (fallback) | Free, active maintenance, A-share complete |
| Dashboard | Streamlit (Phase 16) | Pure Python, built-in charts, single-user |
| Testing | pytest | Standard, inherited from reference project |
| Config | python-dotenv + frozen dataclass | Immutable after load |
| Logging | Python logging with JSON Lines | Debuggable, machine-parseable |