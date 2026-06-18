# Advertising Campaign Optimization Through Funnel Analysis and Experimentation

An end-to-end data analysis project using the Google Analytics Sample Ecommerce Dataset on BigQuery. The goal is to understand user behavior across the purchase funnel, evaluate campaign performance, and produce data-driven recommendations for budget allocation.

---

## Dataset Overview

**Source:** [Google Analytics Sample Dataset](https://console.cloud.google.com/marketplace/product/obfuscated-ga360-data/obfuscated-ga360-data) (BigQuery public data)

**Tables used:**
- `ga_sessions_*` — one table per day, sharded by date (Aug 2016 to Aug 2017, 366 days total)
- `daily_total_visits` — confirmed to match `ga_sessions_*` row counts per day

**Schema highlights:**
- Each row is one session. Nested fields (`hits`, `customDimensions`) require `UNNEST` in SQL.
- Monetary values (`totalTransactionRevenue`, `productPrice`) are stored in micros. Divide by 1,000,000 to get USD.
- `totals.bounces`, `totals.transactions`, and `totals.timeOnSite` use structural NULLs (NULL has meaning, not missing data).
- `campaign` and `city` use `"(not set)"` as a placeholder instead of NULL.
- Pure web dataset: all sessions are pageview-based, screenviews are NULL throughout.

---

## Dataset Statistics

Aggregated across all 366 daily tables (Aug 2016 to Aug 2017):

| Metric | Value |
|---|---|
| Total sessions | 903,653 |
| Unique visitors | 714,167 |
| Avg time on site | 4.4 min |
| Avg hits per session | 4.6 |
| Avg pageviews per session | 3.8 |
| Total transactions | 12,115 |
| Total revenue | $1,780,149 |
| Conversion rate | 1.34% |

**Key distribution findings:**
- Sessions peak in Nov/Dec 2016 (holiday season), with a sharp post-holiday drop in Jan 2017.
- Transaction revenue is heavily right-skewed: median $55.61 vs mean $154.59. A small number of large purchases pull the average up.
- Over 50% of sessions have just 1 pageview. The 90th percentile is 9 pages, reflecting a large low-engagement population alongside a smaller group of highly engaged users.

---

## Project Structure

```
ads-campaign-optimization/
├── notebooks/
│   ├── 01_data_overview.ipynb   # schema, distributions, data quality
│   └── 02_eda.ipynb             # traffic, device, geography, purchase behavior, campaign analysis
├── images/                       # saved chart outputs
├── sql/                          # reusable SQL queries
├── data/                         # gitignored, intermediate outputs
├── credentials/                  # gitignored, BigQuery service account key
├── reports/
├── experiments/
└── dashboard/
```

---

## Key Insights (from EDA)

- **US concentration**: the US generates 94% of transactions from 40% of sessions (3.14% conversion). Canada is the only other market with meaningful traction (0.77%). All other countries convert below 0.16%.
- **Referral is the highest-quality channel**: ~6.25% conversion rate and $718K revenue despite ranking 4th in sessions (after excluding `analytics.google.com`, a non-converting internal traffic source). Social drives reach but almost no revenue (226K sessions, $8K revenue, 0.06% conversion).
- **Engagement predicts purchase**: purchasers view a median of 37 pages vs 2 for non-purchasers. Below ~10 pageviews, almost no one buys.
- **Holiday traffic does not equal holiday revenue**: November had the most sessions ever (114K) but the lowest conversion rate (0.84%). December had the highest conversion (1.83%) but one of the lowest AOVs ($115) - holiday buyers are decisive but spend less per order.
- **Campaign tracking is sparse**: only 3 of 8 named campaigns generated transactions, totalling $28K of $1.78M revenue (<2%). AW - Accessories and AW - Dynamic Search Ads are the only campaigns with measurable return.
- **AOV needs context**: Display ($857), Japan ($424), and April 2017 ($232) all appeared to be high-value segments, but each was driven by a small number of extreme orders. April's true AOV excluding a single dfa/cpm cluster drops from $232 to $137.
- **Mobile barely converts**: Chrome at 1.76% vs Safari at 0.43%. Mobile browsers (Safari in-app, Android Webview) convert below 0.2%.

---

## Phases

- [x] Phase 2: Data Overview (`01_data_overview.ipynb`)
- [x] Phase 3: EDA (`02_eda.ipynb`)
- [ ] Phase 4: Funnel Analysis
- [ ] Phase 5: Campaign Performance
- [ ] Phase 6: Budget Optimization
- [ ] Phase 7: Experiment Design
- [ ] Phase 8: Dashboard
