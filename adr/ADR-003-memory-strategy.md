# ADR-003: Four-Type Memory Strategy with Funnel State

## Status
Proposed

## Context

The system needs to remember four categories of information across runs:

1. **Funnel State**: Where each stock sits in the 3-layer funnel, why it was promoted or eliminated, and when it needs re-evaluation.
2. **Company Snapshots**: The most recent complete analysis for each tracked company -- all indicator values, Agent scores, and the composite report.
3. **Watchlist**: What the user is personally tracking (inherited from ai-investment-mentor).
4. **Decisions**: Every buy/hold/sell recommendation and its outcome (inherited from ai-investment-mentor).

The reference project (ai-investment-mentor ADR-003) established a hybrid strategy: Watchlist is human-curated, Market snapshots are system-managed, Decisions are system-written with human feedback in V2. We inherit this pattern but add a fourth memory type (Funnel) and replace Market Memory with Company Memory.

Key question: who controls the Funnel? Is it fully automated, or does the user have veto power?

## Decision

1. **Funnel Memory is system-managed.** The Screener Agent (deterministic) decides whether a stock passes or fails the hard thresholds. The Chief Investment Agent decides when a stock is promoted from "tracked" (Layer 2) to "deep-researched" (Layer 3). The user cannot override these decisions -- but they can always manually trigger a full analysis on any stock regardless of funnel status.

2. **Company Memory is system-written, immutable history.** Every time a stock is analyzed, the complete snapshot is saved as a new record (never overwritten). This creates an audit trail: "On 2026-Q2 earnings, Quality score was 82. On 2026-Q3, it dropped to 71 because margin compression."

3. **Watchlist Memory is human-curated (inherited).** The system recommends additions/removals but requires user confirmation.

4. **Decision Memory is system-written, human-reviewed (inherited).** The system records every recommendation. In V2, user feedback enables learning.

5. **SQL queries, not vector search (inherited).** V1 uses straightforward SQL lookups. RAG is a V2 consideration when we have enough historical data to justify the complexity.

## Consequences

**What becomes easier:**
- **Funnel auditability.** Every stock's journey through the funnel is recorded: which layer, when it arrived, why it was promoted/eliminated.
- **Quarter-over-quarter comparison.** Company Memory's append-only design means we can diff any two analysis snapshots to see what changed.
- **Zero additional infrastructure.** SQLite handles all four memory types with the same patterns as the reference project.

**What becomes harder:**
- **Funnel staleness.** A stock that passed the Screener 6 months ago may no longer pass today if its financials deteriorated. Without automatic re-screening triggers (Phase 14), this requires manual attention.
- **Schema complexity.** Four memory types means ~8 tables (each type has at least one table, Funnel and Company have related tables for history). More schema to design and maintain.

## Alternatives Considered

### Alternative A: Three memory types (skip Funnel)
Use only Watchlist, Company, and Decision memory. Funnel state is recomputed on every run. Rejected because: (a) the funnel computation involves 6 hard thresholds against 5 years of financial data per stock -- recomputing this for 5300 stocks on every run is wasteful, (b) without persistent funnel state, we lose the ability to track "when did this stock fall out of the funnel and why?"

### Alternative B: Funnel as ephemeral (in-memory only)
Compute funnel state at runtime, don't persist it. Rejected because it prevents historical trend analysis -- "how many stocks passed the Screener in Q1 vs Q2?" is a valuable market health indicator.
