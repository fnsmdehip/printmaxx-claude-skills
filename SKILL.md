# Revenue Tracker

## Overview

Tracks revenue across multiple lanes (apps, digital products, services, content monetization, affiliate) with full attribution from alpha source to asset to revenue. Combines hedge-fund-grade financial analysis (Monte Carlo simulation, Kelly Criterion allocation, Markowitz portfolio optimization) with a CEO feedback loop that drives PROMOTE/KILL decisions on revenue lanes. Designed for portfolio-style operations running 10+ simultaneous revenue methods where no single method should exceed 30% of total revenue.

## Architecture

The revenue tracker is composed of three integrated layers: data collection, financial analysis, and CEO decision feedback.

```
Data Sources                    Financial Engine              CEO Feedback Loop
┌──────────────────┐           ┌─────────────────────┐       ┌──────────────────────┐
│ Stripe            │──┐        │ Revenue Aggregation  │       │ Venture Performance  │
│ (API/webhooks)    │  │        │ - Per-method P&L     │       │ Tracker              │
├──────────────────┤  │        │ - Per-lane totals    │       │ - Lane rankings      │
│ Gumroad           │──┤        │ - Time series        │       │ - Kill triggers      │
│ (sales data)      │  │        ├─────────────────────┤       │ - Double-down trigger│
├──────────────────┤  ├──CSV──▶│ Monte Carlo          │──▶   │                      │
│ App Store/        │  │        │ Projection           │  │   ├──────────────────────┤
│ Play Store        │──┤        │ - 500 simulations    │  │   │ CEO Agent            │
├──────────────────┤  │        │ - Confidence bands   │  └──▶│ - PROMOTE winners    │
│ Affiliate         │──┤        ├─────────────────────┤       │ - KILL losers        │
│ Networks          │  │        │ Kelly Criterion      │       │ - Reallocate capital │
├──────────────────┤  │        │ Capital Allocation   │       │ - Reinvestment matrix│
│ Asset Tracker     │──┘        │ - Optimal bet sizing │       └──────────────────────┘
│ (alpha->asset     │           │ - Risk-adjusted ROI  │
│  attribution)     │           ├─────────────────────┤
└──────────────────┘           │ Portfolio            │
                                │ Optimization         │
                                │ - Diversification    │
                                │ - Correlation matrix │
                                │ - No lane > 30%      │
                                └─────────────────────┘
```

**Data flow:**

1. **Collection** -- revenue data enters the system via CSV trackers. `REVENUE_TRACKER.csv` records per-transaction data: date, method_id, method_name, revenue, expenses, profit, source, notes. `EXPENSE_TRACKER.csv` tracks costs. `ASSET_TRACKER.csv` tracks the alpha-to-asset chain.

2. **Aggregation** -- the financial engine loads CSV data, parses amounts (stripping $, commas), and aggregates by method, lane, and time period. Produces per-method P&L, monthly summaries, and lane-level totals.

3. **Analysis** -- three quantitative analysis engines run on the aggregated data:
   - **Monte Carlo Projection** -- runs 500 simulations of future revenue based on historical distributions. Produces median, P10, P90 confidence bands for 6-12 month projections.
   - **Kelly Criterion Allocation** -- calculates optimal capital allocation per revenue lane based on win rate, average win, and average loss. Prevents over-betting on single methods.
   - **Portfolio Optimization** -- Markowitz-style diversification analysis. Ensures no single lane exceeds 30% of total revenue. Calculates correlation between lanes to maximize risk-adjusted returns.

4. **Attribution chain** -- every revenue dollar is traceable:
   ```
   Alpha entry (ALPHA_STAGING.csv)
     -> approved, venture-tagged
       -> becomes an asset (ASSET_TRACKER.csv: asset_id, alpha_source, asset_type, status, deploy_url)
         -> generates revenue (REVENUE_TRACKER.csv: method_id, revenue, source)
   ```
   If you cannot trace from alpha to asset to revenue, the pipeline is considered broken.

5. **CEO feedback** -- the venture performance tracker reads revenue data and produces lane-level performance rankings. These feed back into the CEO agent's scoring:
   - **Kill trigger**: App < $100 MRR after 60 days. Content account < 500 followers after 90 days. Outbound < 2% reply rate after 3 optimizations.
   - **Double-down trigger**: App MRR growing 20%+ at $500+. Content engagement > 5% sustained 2 weeks. Close rate > 10% on outbound.

**Revenue lanes tracked:**

| Lane | Sources | Kill Threshold | Double-Down Threshold |
|------|---------|---------------|----------------------|
| Apps | App store revenue, in-app purchases, subscriptions | < $100 MRR @ 60d | MRR +20% @ $500+ |
| Digital Products | Gumroad, Whop, course platforms | < $50 in first 30d | > $500/mo sustained |
| Services | Client payments, project fees | < 2% close rate | > 10% close rate |
| Content | Ad revenue, sponsorships, monetized platforms | < 500 followers @ 90d | > 5% engagement sustained |
| Affiliate | Commission payments, referral bonuses | < $25/mo after 60d | > $200/mo sustained |

**Reinvestment matrix (phase-based):**

| Monthly Revenue | Business | Index Funds | Crypto | Angel | Living |
|----------------|----------|-------------|--------|-------|--------|
| $0-1K | 90% | 5% | 5% | 0% | 0% |
| $1K-5K | 80% | 10% | 5% | 0% | 5% |
| $5K-15K | 70% | 10% | 5% | 5% | 10% |
| $15K-50K | 50% | 20% | 10% | 5% | 15% |
| $50K+ | 40% | 25% | 10% | 10% | 15% |

## Required Inputs

- **Revenue tracker CSV** -- per-transaction records with columns: `date`, `method_id`, `method_name`, `revenue`, `expenses`, `profit`, `source`, `notes`
- **Expense tracker CSV** -- cost records with columns: `date`, `category`, `amount`, `description`, `method_id`
- **Asset tracker CSV** -- alpha-to-asset mapping with columns: `asset_id`, `alpha_source`, `asset_type`, `name`, `status`, `deploy_url`, `revenue`, `created_date`, `last_updated`
- **Method master CSV** (optional) -- comprehensive list of all money methods with metadata for cross-referencing

## Outputs

- **Revenue dashboard** -- full financial overview: total revenue, total expenses, net profit, per-method breakdown, monthly trends
- **Per-method P&L** -- individual profit/loss statements for each revenue method
- **Monte Carlo projection** -- 6-12 month revenue forecast with confidence bands (P10, median, P90)
- **Kelly Criterion allocation** -- recommended capital allocation per lane based on risk-adjusted returns
- **Lane performance rankings** -- ordered list of lanes by performance, flagging kill and double-down candidates
- **Kill/double-down recommendations** -- specific recommendations for which lanes to cut and which to scale
- **Attribution report** -- trace from alpha source to asset to revenue for each income stream
- **Reinvestment recommendation** -- based on current revenue level, recommends allocation across business reinvestment, index funds, crypto, and angel investing

## Setup

1. **Create the financial tracking directory:**
   ```
   project/
   ├── FINANCIALS/
   │   ├── REVENUE_TRACKER.csv
   │   ├── EXPENSE_TRACKER.csv
   │   └── P_AND_L_MONTHLY.csv
   ├── LEDGER/
   │   ├── ASSET_TRACKER.csv
   │   └── ALPHA_STAGING.csv
   └── AUTOMATIONS/
       ├── financial_intelligence.py
       └── venture_performance_tracker.py
   ```

2. **Initialize the revenue CSV** with headers:
   ```csv
   date,method_id,method_name,revenue,expenses,profit,source,notes
   ```

3. **Initialize the asset tracker** with headers:
   ```csv
   asset_id,alpha_source,asset_type,name,status,deploy_url,revenue,created_date,last_updated
   ```

4. **Install numpy** (required for Monte Carlo and portfolio optimization):
   ```bash
   pip3 install numpy
   ```

5. **Configure kill/double-down thresholds** in the venture performance tracker for each lane type.

6. **Schedule automated tracking** via cron:
   ```bash
   # Daily P&L update
   0 7 * * * python3 AUTOMATIONS/financial_intelligence.py --pnl
   # Weekly projection
   0 8 * * 1 python3 AUTOMATIONS/financial_intelligence.py --monte-carlo
   ```

## Example Usage

View the full financial dashboard:
```bash
python3 financial_intelligence.py --dashboard
```

Generate monthly P&L:
```bash
python3 financial_intelligence.py --pnl
```

Run 12-month Monte Carlo projection:
```bash
python3 financial_intelligence.py --projection 12
```

Get optimal capital allocation for $5,000:
```bash
python3 financial_intelligence.py --allocate 5000
```

Run Kelly Criterion analysis:
```bash
python3 financial_intelligence.py --kelly
```

View venture performance recommendations:
```bash
python3 venture_performance_tracker.py --recommend
```

## Key Patterns

- **Attribution chain** -- the core data model traces every dollar from its origin (alpha entry) through its manifestation (asset) to its realization (revenue). This chain is enforced as a system invariant: if you cannot trace alpha to asset to revenue, the pipeline is broken. This prevents "ghost revenue" and "ghost assets" that exist in tracking but not in reality.
- **Portfolio-level thinking** -- revenue is never analyzed in isolation. The system views all revenue methods as a portfolio and applies financial portfolio theory: diversification targets, correlation analysis, and concentration limits (no lane > 30% of total).
- **Kill triggers with safety margins** -- kill decisions are based on specific thresholds with time gates. An app is not killed at day 30 for having $80 MRR -- it gets 60 days. This prevents premature kills during ramp-up periods while still enforcing discipline on genuine underperformers.
- **Monte Carlo over linear projection** -- instead of assuming "if we grew 10% last month, we'll grow 10% next month," the system runs 500 random simulations based on historical variance. This produces honest confidence bands rather than optimistic single-line forecasts.
- **Kelly Criterion capital allocation** -- rather than equally splitting capital across lanes, the system calculates mathematically optimal bet sizing based on each lane's win rate and payout ratio. High-conviction, high-win-rate lanes get proportionally more capital.
- **Reinvestment matrix** -- capital allocation changes with scale. At $0-1K/month, 90% goes back into the business. At $50K+/month, only 40% goes back, with the rest distributed across index funds, crypto, and angel investing. This enforces wealth building, not just revenue chasing.
- **Automated P&L** -- financial hygiene is automated, not manual. The system generates P&L statements on schedule, flags stale financial data, and alerts when tracking falls behind.
- **CEO feedback loop** -- revenue data feeds directly into the CEO agent's scoring and decision logic. A lane generating increasing revenue gets a higher score and is more likely to be PROMOTED. A lane with declining revenue and missed thresholds gets KILLED. This closes the loop between execution and strategy.

## Limitations

- **Manual data entry for some sources.** While Stripe and Gumroad can be automated via API, some revenue sources (cash payments, manual transfers) require manual CSV entry. Stale entries = stale analysis.
- **No real-time tracking.** The system processes CSV snapshots on a schedule. There is no live revenue feed or webhook-triggered updates by default.
- **Monte Carlo requires historical data.** With fewer than 30 days of revenue data, the Monte Carlo simulation produces wide confidence bands that are not practically useful. The system becomes more accurate over time.
- **Kelly Criterion assumes independent bets.** In reality, revenue lanes are often correlated (a platform outage affects both content and affiliate revenue). The portfolio optimization partially addresses this, but correlation estimates require significant historical data.
- **No tax or accounting integration.** The system tracks gross revenue and direct expenses but does not handle tax calculations, depreciation, or accounting standards. It is an operational intelligence tool, not accounting software.
- **Attribution can break.** If an asset generates revenue but was not properly linked to its alpha source in the asset tracker, the attribution chain has a gap. The system flags these gaps but cannot automatically repair them.
- **Reinvestment matrix is advisory.** The system recommends allocation percentages but does not execute transactions. Moving money to index funds or crypto requires human action.
