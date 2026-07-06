# Product Vision -- AI Long-Term Investment Analysis System

## Target User

The user is an individual A-share investor who:
- Invests with a 3-10 year horizon -- not a trader, not a day-trader
- Has basic financial literacy (understands PE, ROE, cash flow) but is not a professional analyst
- Currently picks stocks through a mix of: broker recommendations, financial news, personal research on financial statements, and intuition
- Spends several hours per stock on manual research -- reading annual reports, checking indicators, comparing across peers
- Feels that manual analysis is thorough but does not scale: analyzing 10 stocks deeply takes weeks; there are 5300+ stocks they will never get to
- Makes their own final buy/sell decisions -- they do not want a black-box trading bot
- Wants to understand WHY a stock passes or fails each analysis dimension, not just get a final score

## The Problem

Existing tools for Chinese retail long-term investors fall into three categories, and none fully solves the problem:

1. **Data terminals** (Wind, Choice, Eastmoney): Show every number, every indicator, every chart. The user can look up ROE, gross margin, cash flow for any stock. But the cognitive load of (a) knowing which indicators matter, (b) computing them correctly, (c) comparing across peers, (d) tracking changes over quarters -- all of this is on the human. The tools provide data, not analysis.

2. **Broker research reports**: Written by professional analysts, but (a) coverage is concentrated on large-caps -- thousands of mid/small-cap stocks get no coverage, (b) reports carry inherent sell-side bias, (c) research is published on the broker's schedule, not when the user needs it.

3. **AI stock-picking services** (various apps): Deliver conclusions without explanations. "Score: 85/100, Buy." The user cannot evaluate the reasoning, cannot see which dimension drove the score, and cannot adapt the framework to their own investment philosophy.

None of these approaches scale the *analysis* step -- the step where raw financial data becomes a structured evaluation of a company's quality, growth, risk, and valuation. That step currently requires human hours per stock.

## How We Solve It

The AI Long-Term Investment Analysis System sits between raw A-share data and the final investment decision. It does three things:

1. **Computes.** Fetches financial statements (income statement, balance sheet, cash flow) for any A-share stock via AkShare. Computes ~110 indicators across 12 dimensions, applying A-share-specific rules (扣非/归母 ratio, cash collection ratio, goodwill/equity, shareholder pledge ratio, PE/PB percentiles).

2. **Analyzes.** 10 specialized Agents evaluate each dimension independently. Quality Agent assesses profitability trends. Cash Flow Agent verifies profit quality. Risk Agent acts as devil's advocate, applying negative scoring. Every Agent produces a structured score PLUS a qualitative narrative explaining its reasoning.

3. **Explains.** The Chief Investment Agent synthesizes all Agent outputs into a research dossier. Every recommendation carries its complete analysis data: dimension scores, key indicators, Agent narratives, quarter-over-quarter comparison, and a full evidence chain. The user can verify every number and every claim.

The core mechanic: **AI narrows the A-share universe from ~5300 stocks to ~500 screened to ~100 tracked to ~20 deep-researched -- and for every stock that reaches the deep-research stage, the user receives a complete research dossier, not a one-line recommendation.**

This is not a trading bot. It never executes orders. It produces on-demand research dossiers and quarterly portfolio reviews.

## User Journey (V1)

The user has 30 minutes to evaluate whether 贵州茅台 (600519) deserves a position in their long-term portfolio.

1. **Trigger**: They run nalyze 600519 (or the system auto-triggers after Q3 earnings release).

2. **Wait (~60 seconds)**: The system fetches 5 years of financial statements, computes ~110 indicators, runs 10 Agents in parallel, synthesizes the output.

3. **Read the dossier** (5-8 minutes):
   - **At-a-glance**: Composite score 87.3/100, Strong Buy, with dimension breakdown
   - **Quality deep-dive**: ROE trend over 5 years, margin stability, Deducted/Reported NI ratio analysis
   - **Red flags**: Risk Agent found 2 medium-severity risks (policy, macro), no red flags
   - **Valuation context**: PE at 35th percentile of its 5-year history, DCF intrinsic value with margin of safety
   - **Quarter-over-quarter**: What changed since last analysis? Valuation worsened (-5), Growth moderated (-3)
   - **Evidence chain**: Every number traces back to its source (which financial statement, which quarter)

4. **Decide**: The user makes their own buy/hold/skip decision, informed by evidence they can verify.

## Success Criteria (V1)

- A user with basic financial literacy can read a research dossier and articulate WHY a stock scores high or low on each dimension, without memorizing raw data
- Every score in the dossier traces back to either: (a) a computed indicator with a known formula, or (b) an Agent's qualitative analysis with cited evidence
- The analysis pipeline runs end-to-end for a single stock in under 90 seconds
- A user comparing two analysis snapshots (Q2 vs Q3) can identify exactly which indicators changed and by how much

## Non-Goals (What We Do Not Build)

- **Automated trading or order execution.** The human always decides and acts
- **Real-time alerting or streaming.** Analysis runs on trigger (earnings release or manual), not intraday
- **Backtesting or performance benchmarking.** V1 analyzes the present; V2 may add historical validation
- **Multi-user or SaaS.** This is a single-user personal tool
- **Hong Kong, US, or other non-A-share markets.** Scope is mainland China A-shares only
- **Stock screening as a service.** We do not generate daily lists of "top 10 stocks." The user initiates analysis on stocks they are interested in
- **Portfolio optimization or position sizing math.** The Portfolio Agent recommends concentration limits and alerts; it does not compute Markowitz efficient frontiers

## Future Vision (Not V1)

Later phases may add: automatic earnings-release detection and re-analysis (Phase 14), feedback loop where the system tracks its recommendations against actual returns and adjusts confidence (Phase 15), and a Streamlit dashboard for visual portfolio review (Phase 16).
