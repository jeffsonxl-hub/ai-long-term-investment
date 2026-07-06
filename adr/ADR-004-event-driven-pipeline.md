# ADR-004: Event-Driven Pipeline (Not Daily Cron)

## Status
Proposed

## Context

The reference project (ai-investment-mentor ADR-004) chose a DAG-based Pipeline executed daily via cron at 8:30 AM. This makes sense for a daily morning report system where market conditions change every trading day.

Our system has fundamentally different time scales:

- Financial statements update quarterly (not daily)
- Company quality (ROE, margins, moat) changes over years, not days
- Valuation changes daily but a long-term investor should not re-evaluate daily -- that defeats the purpose
- Running a full 9-Agent analysis on 500 stocks every day would be wasteful (~4000 LLM calls/day for questionable value)

We need a different trigger model.

## Decision

We will implement an **event-driven Pipeline** with three trigger types:

1. **Earnings Release Trigger (primary).** When a tracked company releases quarterly/annual results, the Indicator Engine recomputes all indicators and all Agents re-score the company. This is the natural rhythm of long-term analysis.

2. **Manual Trigger (user-initiated).** The user can request a full analysis on any stock at any time: "analyze 600519" or "analyze portfolio."

3. **Watchdog Trigger (lightweight, daily).** The Risk Agent runs a lightweight scan on all tracked stocks checking for Red Flag events: regulatory penalties, majority shareholder pledge changes > 10%, audit opinion changes, ST warnings. No full LLM analysis -- just deterministic rules + news scanning.

The Pipeline code itself (DAG executor with asyncio, step dependencies, severity model, retry logic) is inherited from ai-investment-mentor ADR-004. Only the trigger model changes.

## Consequences

**What becomes easier:**
- **Cost efficiency.** LLM calls only happen when there is new information to analyze (earnings release) or when the user explicitly asks. No wasted daily runs.
- **Analysis depth.** Because runs are infrequent, each run can be more thorough -- longer LLM context windows, more evidence to evaluate.
- **Natural rhythm.** Quarterly analysis matches how professional long-term investors actually work.

**What becomes harder:**
- **Earnings release detection.** We need a calendar of expected release dates + a mechanism to detect actual releases. AkShare provides this, but it is one more integration.
- **Stale data risk.** If earnings release detection fails, a company may go unreviewed for months. Mitigation: Watchdog Trigger can flag "no new analysis in 120 days" as a low-severity alert.
- **Batch scheduling.** During earnings season (4-6 weeks of concentrated releases), the system may need to process 50+ companies in a short window. Mitigation: queue-based processing with configurable parallelism.

## Alternatives Considered

### Alternative A: Daily cron (like ai-investment-mentor)
Run the full pipeline every trading day. Rejected because: (a) long-term fundamentals don't change daily, (b) daily LLM analysis of the same company with nearly identical data would produce nearly identical reports -- wasteful, (c) daily reports would train the user to check daily, which is exactly the behavior long-term investing should discourage.

### Alternative B: Manual-only (no auto-trigger)
User must explicitly request every analysis. Rejected because: (a) the user may miss earnings releases, (b) one of the system's value propositions is that it monitors things the user would miss.

### Alternative C: Scheduled quarterly
Run on fixed calendar dates (March 31, June 30, etc.). Simpler than earnings-release detection but: (a) companies release on different schedules, (b) some release late -- running on a fixed date may analyze before the data is available.
