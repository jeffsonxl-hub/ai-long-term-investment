# Agent Architecture -- AI Long-Term Investment Analysis System

This document answers: **Why these Agents? How do they relate? What does each one own?**

It is the single source of truth for how intelligence is organized in this system.

---

## Core Design Question

A long-term investment system needs to answer fundamentally different questions than a daily trading system:

| Question | Our Answer | How |
|----------|-----------|-----|
| Is this company profitable? | Quality Agent | ~20 indicators from financial statements |
| Is it growing sustainably? | Growth Agent | ~10 indicators, historical + projected |
| Is the profit real? | Cash Flow Agent | ~10 indicators, cross-validates income statement |
| Is it cheap enough to buy? | Valuation Agent | ~10 indicators + DCF, PE/PB percentiles |
| Will it survive (not blow up)? | Financial Agent | ~15 indicators, A-share-specific red flags |
| Is management trustworthy? | Governance Agent | ~10 indicators, qualitative + quantitative |
| Is the industry tailwind or headwind? | Industry Agent | ~10 indicators, policy + competitive dynamics |
| What could go wrong? | Risk Agent | ~10 indicators, negative-scoring only |

This is fundamentally different from a daily-trading system (ai-investment-mentor) which asks: "What regime are we in? What themes have momentum? What news just broke?"

We need more Agents because the analysis dimensions are more numerous and more independent of each other. A company can have great profitability but terrible governance. The market may love it (low Risk score) but the industry may be dying (low Industry score). These dimensions should not be conflated in a single Agent's reasoning pass.

---

## Architecture: Advisor-Centric (Hub-and-Spoke)

`
                         User (Investor)
                              |
                    Chief Investment Agent
                    (orchestrates, synthesizes,
                     produces final report)
                              |
        +---------------------+---------------------+
        |                     |                     |
   Screening Layer       Analysis Layer        Context Layer
   (Stage 1: Funnel)    (Stage 2: Deep)      (Stage 3: Environment)
        |                     |                     |
   Screener Agent       Quality Agent         Industry Agent
   (deterministic,       Growth Agent          Macro Agent
    no LLM)             Cash Flow Agent
                        Valuation Agent
                        Financial Agent
                        Governance Agent
                        Risk Agent
`

**All arrows point inward.** Specialist Agents never call each other. The Chief Investment Agent is the sole orchestrator. This is the same topology as ai-investment-mentor's Advisor-centric architecture (ADR-001), adapted for our larger Agent count.

---

## Agent Catalog

### Screen 1: Screener Agent (Funnel Layer 1)

| Attribute | Value |
|-----------|-------|
| **LLM?** | No. Pure deterministic rules. |
| **Purpose** | Narrow 5300+ A-shares to ~500 that pass hard thresholds. |
| **Rules** | ROE 5yr > 15%, OCF/NI > 0.7, Goodwill/Equity < 20%, Pledge < 30%, no ST, clean audit |
| **Output** | Pass/Fail list with violation reasons for fails. |
| **Memory** | Writes to Funnel Memory: which stocks passed, which failed, and why. |
| **Runs** | Quarterly (after earnings season). |

**Why deterministic?** These rules have no ambiguity. ROE is either > 15% or not. An LLM adds latency and hallucination risk with zero benefit.

---

### Screen 2: Analysis Layer -- 7 Specialist Agents

Each Agent receives a stock code + its pre-computed indicators (from the Indicator Engine in Stage 2 of the roadmap), produces a structured analysis, and writes to Company Memory. They run in parallel for a given stock.

#### Quality Agent

| Attribute | Value |
|-----------|-------|
| **Identity** | A disciplined fundamental analyst. You examine profitability like Warren Buffett -- long track record, no shortcuts. |
| **Purpose** | Assess whether this company consistently earns high returns on capital. |
| **Inputs** | ROE (5yr), ROA, ROIC, Gross Margin, Net Margin, EBIT Margin, EBITDA Margin, Deducted/Reported NI ratio, DuPont decomposition |
| **Output** | Quality Score (0-100) + breakdown: Profitability (60%) + Efficiency (25%) + Market Position (15%) |
| **LLM Role** | Evaluates moat width, competitive durability, and whether margin trends are structural or cyclical. |
| **A-Share Special** | Deducted/Reported NI ratio analysis -- flags companies where reported profit significantly exceeds core operating profit. |

#### Growth Agent

| Attribute | Value |
|-----------|-------|
| **Identity** | A growth analyst who distinguishes real compounding from one-off spikes. |
| **Purpose** | Assess historical consistency and future trajectory of earnings growth. |
| **Inputs** | Revenue/Profit/EPS CAGR (3yr, 5yr), FCF growth, R&D/Revenue trend, analyst consensus |
| **Output** | Growth Score (0-100) + breakdown: Historical (60%) + Future Outlook (30%) + Quality (10%) |
| **LLM Role** | Evaluates whether growth is sustainable -- industry tailwind vs. one-time event, R&D pipeline quality. |
| **A-Share Special** | Revenue vs. profit growth divergence flag -- profit growing faster than revenue often means cost-cutting or non-recurring gains, not real growth. |

#### Cash Flow Agent

| Attribute | Value |
|-----------|-------|
| **Identity** | A cash flow detective. Profit can be manufactured; cash receipts cannot. |
| **Purpose** | Verify that reported profit translates into actual cash received. |
| **Inputs** | OCF, FCF, OCF/NI ratio, Cash Collection Ratio, FCF Margin, CapEx/OCF, OCF growth |
| **Output** | Cash Flow Score (0-100) + breakdown: Quality (50%) + Generation (30%) + Growth (20%) |
| **LLM Role** | Explains divergence between profit and cash flow -- is it aggressive revenue recognition, working capital build, or legitimate investment? |
| **A-Share Special** | Cash Collection Ratio (< 0.8 = severe warning). This is the single best A-share fraud detection indicator. |

#### Valuation Agent

| Attribute | Value |
|-----------|-------|
| **Identity** | A valuation expert who uses multiple models to triangulate fair value. |
| **Purpose** | Determine whether the current price offers an adequate margin of safety. |
| **Inputs** | PE (TTM), PE percentile, Forward PE, PB, PB percentile, PEG, EV/EBITDA, Dividend Yield, DCF inputs |
| **Output** | Valuation Score (0-100) + DCF intrinsic value + Margin of Safety + percentile analysis |
| **LLM Role** | Explains why valuation is where it is -- structural discount, cyclical trough, or genuine bargain. |
| **A-Share Special** | PE/PB percentiles (not absolute values). A-share valuation中枢 has been declining for years -- a PE of 20 may be expensive for one sector and cheap for another. |

#### Financial Health Agent

| Attribute | Value |
|-----------|-------|
| **Identity** | A balance sheet auditor. Your job is to find out if this company could go bankrupt. |
| **Purpose** | Assess solvency, liquidity, and balance sheet quality. |
| **Inputs** | Debt/Asset, Interest-bearing Debt/Asset, Current Ratio, Quick Ratio, Interest Coverage, Debt/EBITDA, Net Cash, Goodwill/Equity, Pledge Ratio, AR/Revenue trend, Inventory/Revenue trend |
| **Output** | Financial Health Score (0-100) + Red Flag list |
| **LLM Role** | Evaluates whether debt structure is reasonable for the business model -- some industries legitimately carry high leverage. |
| **A-Share Special** | Goodwill/Equity > 30% = severe warning. Pledge > 50% = near-automatic disqualification. These are A-share-specific landmines with no direct US equivalent. |

#### Governance Agent

| Attribute | Value |
|-----------|-------|
| **Identity** | A corporate governance investigator. You read between the lines of regulatory filings. |
| **Purpose** | Assess management quality, alignment with shareholders, and regulatory compliance. |
| **Inputs** | Insider trading records, executive turnover, related-party transactions, regulatory penalties, institutional holdings (Social Security Fund + Northbound), ESG data |
| **Output** | Governance Score (0-100) + Management Alert List |
| **LLM Role** | Qualitative analysis of management's historical decision-making, capital allocation track record, and shareholder communication quality. |
| **A-Share Special** | Social Security Fund and Northbound holdings as quality signals -- these institutions have superior research and their sustained buying is a strong vote of confidence. |

#### Risk Agent

| Attribute | Value |
|-----------|-------|
| **Identity** | The Devil's Advocate. Your sole job is to find reasons NOT to invest. |
| **Purpose** | Identify all material risks and apply negative scoring. |
| **Inputs** | All indicators from above Agents + external risk signals (regulatory, industry disruption, macro vulnerability) |
| **Output** | Risk Score (0-100, where higher = riskier) + Categorized Risk Register |
| **LLM Role** | Synthesizes cross-dimensional risk -- e.g., "high debt + industry downturn = amplified downside." |
| **Scoring** | **Negative-only.** Risk Agent never adds points. It only subtracts. A perfect score means "no red flags found," not "this is a safe stock." |

---

### Screen 3: Context Layer -- 2 Environment Agents

These Agents analyze the external environment, not individual stocks. They run once per analysis cycle (not per stock).

#### Industry Agent

| Attribute | Value |
|-----------|-------|
| **Identity** | An industry strategist who maps competitive dynamics. |
| **Purpose** | Assess whether a company's industry is a tailwind or headwind for long-term returns. |
| **Inputs** | Industry growth rate, CR3/CR5, policy direction, capacity utilization, global competitiveness, technology disruption risk |
| **Output** | Industry Score (0-100) + Industry Outlook narrative |
| **LLM Role** | Porter Five Forces analysis, policy direction interpretation, global competitive positioning. |

#### Macro Agent

| Attribute | Value |
|-----------|-------|
| **Identity** | A macroeconomist watching the big picture. |
| **Purpose** | Assess whether the macro environment supports or threatens the investment thesis. |
| **Inputs** | GDP growth, CPI/PPI, LPR/rates, M1/M2, social financing, RMB exchange rate, commodity prices |
| **Output** | Macro Score (0-100) + Macro Narrative |
| **LLM Role** | Interprets policy signals and macro trends in the context of specific industries. |

---

### Chief Investment Agent (Orchestrator)

| Attribute | Value |
|-----------|-------|
| **Identity** | The investment committee chair. You do no original research -- you synthesize, challenge, and decide. |
| **Purpose** | Orchestrate all specialist Agents, produce a composite analysis and final investment thesis. |
| **Inputs** | All Agent scores + reports + Funnel Memory + Decision Memory |
| **Output** | Composite Score (0-100 weighted) + Investment Thesis Report + Portfolio Recommendation |
| **LLM Role** | Heavy usage. Synthesizes quantitative scores with qualitative narratives. Identifies contradictions between Agents (e.g., high Quality but high Risk). Generates the final readable report. |
| **Constraints** | Never accesses data sources directly. Never overrides a Risk Agent Red Flag without explicit justification. Every recommendation cites at least 3 independent evidence sources. |

---



### Output Contract (Chief Investment Agent)

The Chief produces a Research Dossier per stock. This is NOT a one-paragraph
recommendation -- it is the complete analysis package: narrative, scores,
evidence, and the raw data that drove them. The user can verify every claim.

`json
{
  "report_id": "rpt-2026-Q3-600519",
  "generated_at": "2026-11-01T09:30:00+08:00",
  "trigger": "earnings_release",
  "stock": {
    "code": "600519",
    "name": "Kweichow Moutai",
    "sector": "Food & Beverage",
    "market_cap_bn": 2100.5
  },

  "funnel_status": {
    "layer": 3,
    "passed_screener": true,
    "screener_date": "2025-04-28",
    "promoted_to_deep": "2025-10-15"
  },

  "composite_score": {
    "total": 87.3,
    "rating": "Excellent",
    "action": "Strong Buy",
    "breakdown": {
      "quality": {"score": 92, "weight": 0.20},
      "growth":  {"score": 78, "weight": 0.15},
      "cash_flow": {"score": 95, "weight": 0.15},
      "valuation": {"score": 72, "weight": 0.15},
      "financial_health": {"score": 88, "weight": 0.10},
      "industry": {"score": 85, "weight": 0.08},
      "risk": {"score": 82, "weight": 0.07},
      "governance": {"score": 90, "weight": 0.05},
      "macro": {"score": 70, "weight": 0.03},
      "technical": {"score": 65, "weight": 0.02}
    },
    "penalties_applied": []
  },

  "analysis_data": {
    "quality": {
      "agent": "Quality Agent",
      "narrative": "Moutai maintains exceptional profitability...",
      "key_indicators": {
        "roe_5yr_avg": 0.31,
        "roe_trend": "stable",
        "roic_5yr_avg": 0.33,
        "gross_margin_5yr_avg": 0.92,
        "gross_margin_trend": "improving",
        "net_margin_5yr_avg": 0.52,
        "deducted_vs_reported_ni_ratio": 0.98,
        "ebit_margin": 0.68
      }
    },
    "growth": {
      "agent": "Growth Agent",
      "narrative": "Revenue growth has moderated to...",
      "key_indicators": {
        "revenue_cagr_3yr": 0.12,
        "revenue_cagr_5yr": 0.16,
        "net_profit_cagr_3yr": 0.14,
        "net_profit_cagr_5yr": 0.19,
        "eps_cagr_3yr": 0.14,
        "revenue_vs_profit_divergence": false,
        "rd_to_revenue_trend": "stable"
      }
    },
    "cash_flow": {
      "agent": "Cash Flow Agent",
      "narrative": "Cash conversion is excellent...",
      "key_indicators": {
        "ocf_to_ni_5yr_avg": 1.12,
        "fcf_positive_years": 5,
        "cash_collection_ratio": 1.08,
        "fcf_margin": 0.48,
        "capex_to_ocf": 0.08
      }
    },
    "valuation": {
      "agent": "Valuation Agent",
      "narrative": "Currently trading at 28x PE...",
      "key_indicators": {
        "pe_ttm": 28.3,
        "pe_percentile_5yr": 0.35,
        "pe_percentile_10yr": 0.42,
        "pb": 9.2,
        "pb_percentile_5yr": 0.38,
        "peg": 1.8,
        "ev_ebitda": 22.1,
        "dividend_yield": 0.018,
        "dcf_intrinsic_value": 1950.0,
        "current_price": 1680.0,
        "margin_of_safety": 0.14
      }
    },
    "financial_health": {
      "agent": "Financial Health Agent",
      "narrative": "Balance sheet is fortress-grade...",
      "key_indicators": {
        "debt_to_asset": 0.18,
        "interest_bearing_debt_ratio": 0.02,
        "current_ratio": 3.8,
        "quick_ratio": 2.9,
        "interest_coverage": 85.0,
        "debt_to_ebitda": 0.3,
        "net_cash": 98000000000,
        "goodwill_to_equity": 0.0,
        "pledge_ratio": 0.0,
        "ar_to_revenue_trend": "stable",
        "inventory_to_revenue_trend": "stable"
      }
    },
    "governance": {
      "agent": "Governance Agent",
      "narrative": "Management is stable with strong alignment...",
      "key_indicators": {
        "insider_selling_1yr": "none",
        "executive_turnover_5yr": "low",
        "related_party_tx_ratio": 0.01,
        "regulatory_penalties": "none",
        "social_security_fund_holding": "increasing",
        "northbound_holding_trend": "increasing"
      }
    },
    "risk": {
      "agent": "Risk Agent",
      "narrative": "Primary risks are macro and policy...",
      "red_flags": [],
      "risk_register": [
        {"category": "policy", "severity": "medium", "detail": "Alcohol consumption tax reform under discussion"},
        {"category": "macro", "severity": "medium", "detail": "Consumer spending slowdown affects premium liquor demand"}
      ]
    },
    "industry": {
      "agent": "Industry Agent",
      "narrative": "Baijiu industry remains structurally attractive...",
      "key_indicators": {
        "industry_growth_cagr_3yr": 0.08,
        "cr3": 0.45,
        "cr5": 0.58,
        "policy_direction": "neutral",
        "capacity_utilization": 0.85,
        "company_market_share": 0.18,
        "moat_type": "brand"
      }
    },
    "macro": {
      "agent": "Macro Agent",
      "narrative": "Moderate GDP growth with accommodative monetary policy...",
      "key_indicators": {
        "gdp_growth": 0.048,
        "cpi": 0.018,
        "lpr_1yr": 0.0345,
        "m2_growth": 0.085,
        "rmb_usd": 7.28
      }
    }
  },

  "investment_thesis": {
    "summary": "Moutai remains the highest-quality A-share company with unparalleled brand moat, 92% gross margins, and cash conversion above 100%. Current valuation at 28x PE (35th percentile) offers a moderate but not exceptional entry point. The investment case rests on: (1) pricing power -- continued ability to raise ex-factory prices, (2) direct sales channel expansion improving margin mix, (3) dividend policy improving shareholder returns.",
    "catalyst": "Potential ex-factory price increase in 2027. Direct sales ratio rising from 45% to 55% over 2 years.",
    "risk_scenario": "If anti-corruption campaign intensifies or alcohol tax increases materially, premium baijiu demand could contract 15-20%. In this scenario, PE could compress to 22x (historical trough), implying ~20% downside.",
    "invalidation": "Gross margin decline below 88% OR revenue growth turns negative for 2 consecutive quarters."
  },

  "comparison": {
    "vs_previous_quarter": {
      "composite_score_change": -2.1,
      "key_changes": [
        {"dimension": "valuation", "change": -5, "reason": "PE expanded from 26x to 28x as price rose 8%"},
        {"dimension": "growth", "change": -3, "reason": "Revenue CAGR 3yr declined from 14% to 12%"},
        {"dimension": "governance", "change": +2, "reason": "Social Security Fund increased position"}
      ]
    },
    "vs_industry_median": {
      "quality_percentile": 98,
      "valuation_percentile": 45,
      "growth_percentile": 55
    }
  },

  "evidence_chain": [
    {"source": "Indicator Engine", "type": "computed", "detail": "ROE 5yr avg = 31% from income statement + balance sheet"},
    {"source": "Quality Agent", "type": "llm_analysis", "detail": "Margin trend analysis based on 20 quarters of data"},
    {"source": "Cash Flow Agent", "type": "computed", "detail": "Cash collection ratio = 1.08 from cash flow statement"},
    {"source": "Valuation Agent", "type": "computed", "detail": "DCF intrinsic value = 1950 based on 3-stage growth model"},
    {"source": "Risk Agent", "type": "llm_analysis", "detail": "Policy risk assessment based on regulatory news scan"}
  ]
}
`

### Design Rule: No Naked Recommendations

Every stock that appears in a report or portfolio MUST carry its complete
nalysis_data package. If the user asks "why is 600519 rated Strong Buy?"
the answer is: (1) the composite score breakdown showing every dimension,
(2) the key indicators that drove each dimension score, (3) the Agent
narratives explaining qualitative judgments, (4) the evidence chain tracing
every claim back to source data.

This is the difference between a recommendation and a research dossier.

## Information Flow (Single Stock Analysis)

`
Trigger: User requests analysis of 600519
    |
Step 1: Indicator Engine computes all ~80 indicators (deterministic)
    |
Step 2: Screener Agent verifies funnel passage (deterministic)
    |
Step 3: 7 Analysis Agents run IN PARALLEL
    |       Quality    Growth    CashFlow    Valuation    Financial    Governance    Risk
    |       (each reads indicators, calls LLM for qualitative analysis,
    |        returns structured score + narrative)
    |
Step 4: 2 Context Agents provide industry + macro context
    |       (run once per batch, shared across all stocks being analyzed)
    |
Step 5: Chief Investment Agent synthesizes all outputs
    |       - Weighted composite score
    |       - Cross-Agent contradiction detection
    |       - Narrative investment thesis
    |       - Final recommendation (Strong Buy / Buy / Hold / Watch / Avoid)
    |
Step 6: Output -- Research Report saved to Company Memory + delivered to user
`

---

## Why This Agent Count?

The reference project (ai-investment-mentor) has 4 specialist Agents for daily trading analysis. Our domain requires more because:

1. **Analysis dimensions are more independent.** In daily trading, "market regime" and "news sentiment" and "technical momentum" often converge on the same signal. In long-term investing, "profitability" and "governance" and "valuation" are genuinely orthogonal -- a company can score 95 on Quality and 20 on Governance, and that tension IS the insight.

2. **A-share-specific risks need dedicated attention.** Goodwill impairment, majority shareholder pledging, and deducted-vs-reported profit divergence are A-share phenomena. Bundling them into a generic "Risk" bucket would dilute their importance. Financial Health Agent and Governance Agent exist specifically to spotlight these.

3. **LLM reasoning is domain-specific.** A single LLM prompt trying to evaluate ROE trends, management integrity, industry policy direction, AND valuation percentile all at once produces shallow analysis. Dedicated Agents with focused prompts produce deeper, more auditable output.

---

## Comparison with ai-investment-mentor

| Aspect | ai-investment-mentor | ai-long-term-investment |
|--------|---------------------|------------------------|
| Agent count | 4 specialist + 1 advisor | 9 specialist + 1 chief |
| Pipeline trigger | Daily cron | Event-driven (earnings release) |
| Core intelligence | Market regime classification | Fundamental quality scoring |
| LLM usage | Heavy in Advisor (narrative) | Light in specialists (qualitative), heavy in Chief (synthesis) |
| Scoring | 4-factor, regime-weighted | 12-dimension, purpose-weighted |
| Risk handling | Implicit in scoring | Explicit negative-only Risk Agent |
| Memory types | 3 (Watchlist, Market, Decision) | 4 (+ Funnel Memory) |
