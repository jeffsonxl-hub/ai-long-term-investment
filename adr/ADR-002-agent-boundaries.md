# ADR-002: Agent Responsibilities and Data Access Boundaries

## Status
Proposed

## Context

With 9 specialist Agents and one Chief Investment Agent, we need clear rules about what each Agent is allowed to do -- and what it is explicitly forbidden from doing. Without boundaries, an Agent will naturally expand its scope, duplicating logic and creating confusion about where truth lives.

Two specific boundary questions emerged:

1. **Should the Chief Investment Agent be allowed to access the Indicator Engine directly?** It is tempting: if the Chief wants to verify a stock's ROE, why force it to wait for the Quality Agent? The answer is that direct data access erodes traceability -- a composite score cannot be audited if the Chief fetched data outside the structured evidence chain.

2. **Should the Indicator Engine (deterministic computation) live in the Agent Layer or the Tool Layer?** The Indicator Engine has no LLM, no reasoning, no decisions. It computes formulas from financial statements. By the Agent-vs-Component rule, it belongs in the Tool Layer. But it is the most critical computation in the entire system -- placing it in Tools with 30-second timeouts and retry logic doesn't reflect its importance.

## Decision

1. **The Chief Investment Agent never accesses the Indicator Engine or Data Layer directly.** All data reaches the Chief through structured output from specialist Agents. The Chief works with scores and narratives, not raw indicator values.

2. **The Indicator Engine is a Component, not a Tool.** It has no timeout, no retry, no network access. It computes ~80 indicators from pre-fetched financial data. It is called BEFORE any Agent runs, and its output is passed to all Agents as input. This elevates it above the Tool Layer's "best-effort" semantics.

3. **Specialist Agents never call each other.** The Chief is the sole orchestrator. If the Growth Agent needs revenue data, the Chief provides it as an input parameter from the Indicator Engine output.

4. **Every Agent accesses Memory only through MemoryRepository.** No Agent opens a database connection, writes SQL, or manages file handles.

5. **The Screener Agent is the only Agent that can eliminate a stock from the pipeline.** If Screener fails a stock, no downstream Agent sees it. This prevents LLM Agents from "talking themselves into" a stock that fails hard thresholds.

## Consequences

**What becomes easier:**
- **Auditability.** Open any composite score: the evidence trail is Indicator Engine -> [Specialist Agent] -> Chief. No side channels exist.
- **Replaceability.** We can swap the Valuation Agent's DCF model without touching any other Agent.
- **Parallel development.** Two Agents can be built independently as long as they respect the input/output contract.
- **Indicator Engine reliability.** As a Component, it runs once, deterministically, and its output is shared. No Agent re-computes indicators from scratch.

**What becomes harder:**
- **Orchestration overhead.** The Chief must explicitly gather all indicator data and Agent outputs before synthesizing. More boilerplate.
- **Data duplication.** The same Indicator Engine output is passed to 8 Agents. Each receives the full ~80-indicator dict but only uses ~10 relevant ones.
- **Latency from serialization.** Inter-Agent data passes through JSON serialization. For a single stock this is negligible. For batch analysis of 500 stocks, this matters.

## Alternatives Considered

### Alternative A: Chief with direct Indicator Engine access
The Chief could call get_roe("600519") directly. Rejected because it creates two paths to the same data and makes it impossible to audit whether a score came from specialist analysis or an undisclosed direct lookup.

### Alternative B: Indicator Engine as a Tool
Place Indicator computation in the Tool Layer alongside AkShare data fetching. Rejected because: (a) Tools have timeouts and retry semantics appropriate for network calls, not pure computation, (b) Indicator Engine is the foundation every Agent depends on -- treating it as a fallible Tool would mean every Agent needs indicator-failure handling logic.

### Alternative C: Shared data context object (like LangGraph State)
All Agents read from a shared in-memory context. Elegant for LangGraph (Phase 12 of ai-investment-mentor) but for V1 with a simple Python orchestrator, a shared mutable state introduces debugging nightmares. Rejected for V1 -- revisit when we consider LangGraph migration.
