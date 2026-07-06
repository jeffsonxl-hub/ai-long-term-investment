# ADR-001: Advisor-Centric (Hub-and-Spoke) Architecture

## Status
Proposed

## Context

We need to decide how intelligence is organized. The system must analyze ~110 indicators across 12 dimensions, produce quantitative scores and qualitative narratives, and synthesize everything into a readable investment thesis for a single stock or a portfolio.

The core question: should these analysis capabilities be implemented as one monolithic Agent (one LLM call that does everything), or as a Chief Investment Agent that orchestrates specialist Agents?

A monolithic Agent would be simpler -- one prompt, one output, fewer interfaces. But mixing profitability analysis, cash flow verification, governance assessment, and industry positioning in one reasoning pass makes it hard to debug why a particular score or recommendation was produced. When a score changes quarter-over-quarter, you cannot trace which dimension drove the change.

The reference project (ai-investment-mentor) faced the same question and chose Advisor-centric (ADR-001). We inherit the same reasoning, but our domain has more analysis dimensions, making the case for decomposition even stronger.

## Decision

We will use a **Chief Investment Agent as orchestrator** with 9 specialist Agents (Screener, Quality, Growth, Cash Flow, Valuation, Financial, Governance, Risk, Industry, Macro). The Screener is deterministic (no LLM). The other 8 specialist Agents each own one analysis dimension. The Chief Investment Agent is the sole orchestrator -- it gathers structured output from all specialists and synthesizes the final composite score and investment thesis.

**Core rule**: Specialist Agents never call each other. All coordination flows through the Chief Investment Agent.

`
Specialist Agents (produce dimension-specific analysis)
       |
       v
Chief Investment Agent (synthesizes + weights + explains)
       |
       v
     User
`

## Consequences

**What becomes easier:**
- **Debuggability.** Every score traces back to a specific Agent. If the Quality score drops from 85 to 70, we know exactly which Agent to investigate.
- **Independent iteration.** We can improve the Cash Flow Agent's fraud detection logic without touching the Valuation Agent.
- **Parallel execution.** All 7 Analysis Agents run simultaneously for a given stock. Pipeline latency is capped by the slowest Agent, not the sum of all Agents.
- **Testing.** Each Agent can be tested in isolation with mock indicator data.
- **Prompt quality.** Each Agent has a focused, domain-specific prompt. A single "analyze this stock" prompt would be too broad to produce deep analysis.

**What becomes harder:**
- **Interface discipline.** We must define structured data contracts between 10 Agents. The score and narrative format must be consistent.
- **Orchestration complexity.** The Chief Investment Agent must gather, validate, and weight 8 separate inputs before synthesizing.
- **LLM cost.** 8 parallel Agent runs means up to 8 LLM calls per stock analysis. Mitigation: deterministic pre-filtering (Screener eliminates ~90% of stocks before any LLM is called).
- **Cross-dimension insight.** If the Quality Agent detects a margin decline and the Industry Agent detects competitive pressure, the connection between these observations is only made at the Chief level. A monolithic Agent might catch this earlier.

## Alternatives Considered

### Alternative A: Monolithic Agent
One Agent receives all indicators and produces a single analysis. Rejected for the same reasons as ai-investment-mentor: (a) debugging is nearly impossible -- which indicator drove the score change? (b) a single prompt that must reason about profitability AND governance AND industry dynamics produces shallow analysis, (c) the system is harder to extend -- adding a new dimension means rewriting the entire prompt.

### Alternative B: Peer-to-Peer Agents
Agents communicate directly (Quality tells Growth what it found, Growth adjusts its analysis). Rejected because: (a) this creates circular dependencies and makes testing impossible without the full system running, (b) it violates the "single responsibility" rule -- if Quality Agent's output format changes, Growth Agent breaks too.

### Alternative C: Pipeline-only (no LLM Agents)
Pure deterministic scoring with no qualitative analysis. All 110 indicators feed into a weighted formula. Rejected because: (a) governance, moat, and industry dynamics genuinely require qualitative reasoning, (b) the user explicitly wants explanations, not just scores, (c) the reference project's core insight is that LLMs add value in synthesis and explanation.
