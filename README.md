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
│   ├── 02_eda.ipynb             # traffic, device, geography, purchase behavior, campaign analysis
│   └── 03_funnel_analysis.ipynb # purchase funnel drop-off by device, channel, country, visitor type, and time
├── images/                       # saved chart outputs
├── sql/
│   └── funnel_analysis.sql       # all BigQuery queries from 03_funnel_analysis.ipynb
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

## Key Insights (from Funnel Analysis)

- **86% of sessions never see a product page.** The biggest loss in the funnel is at discovery, not checkout. Only 1.30% of all sessions result in a purchase.
- **Checkout friction is at Payment and Review, not the confirm button.** 26% drop off entering payment details and 29% at the review screen; only 1.4% abandon once they reach the final confirm step.
- **Mobile converts at 4x lower rate than desktop** (0.41% vs 1.62%). The gap opens at Add to Cart and holds through checkout - a UX friction problem, not a reach problem.
- **Referral converts best because it enters the funnel deeper and drops off less.** 29.5% of Referral sessions reach a product page vs 16.4% for Organic Search. Social drives volume (226K sessions) but 98% never click into a product.
- **International traffic faces a checkout wall.** US converts at 3.04% vs Rest of World at 0.11% (28x gap). International users who do reach checkout drop off at 89% before completing - a signal of shipping restrictions or unsupported payment methods.
- **Returning visitors are 22% of sessions but 61% of purchases.** They convert at 3.66% vs 0.65% for new visitors (6x gap), and their drop-off rate declines at every funnel step - the deeper they go, the more committed they become.
- **Best time to convert: Monday and Friday, 10 AM - 2 PM Pacific.** Purchase rate peaks at 2 PM (2.02%) and weekday intent (1.35-1.49%) is nearly double weekend rates (0.81-0.96%).

---

## Key Insights (from Product Analysis)

- **Revenue is concentrated but not extreme:** the top 20% of products (98 SKUs) account for 69.4% of revenue; the bottom 50% contribute only 7.5%. The top 49 products deliver 51% of revenue - a manageable set to prioritize for stock and promotion.
- **Apparel dominates revenue, Office dominates volume:** Apparel accounts for 39.3% of total revenue ($684K). Office leads in units sold (94,669 - nearly 3x Apparel) at a low effective price of $3.78/unit, driven by bulk journal and notebook orders that signal corporate gifting.
- **Bags and Electronics have a view-to-cart problem (~18%):** both categories draw high traffic but rarely convert to add-to-cart. Accessories convert best into the cart (41%) but worst out of it (22%) - impulse adds that get abandoned if the session does not result in a primary purchase.
- **Google brand has a consistent purchase conversion advantage:** in matched-pair comparisons (same product, different brand), Google cart-to-purchase exceeds YouTube in all 4 pairs and Android in 3 of 4. Google converts at 2x YouTube's rate on journals even when priced higher, indicating strong brand-driven pricing power.
- **Two drivers of high revenue per detail-view session:** high unit price (Vest at $65.66/unit, Headphones at $47.86/unit generate meaningful revenue even with modest purchase rates) and bulk buying (22 oz Water Bottle averages 5.78 units/session, Recycled Paper Journal Set 5.15 units/session). The highest-traffic high-yield products - 26 oz Double Wall Bottle (3,424 views, $11.67/session) and Leatherette Journal (1,713 views, $15.47/session) - are the most efficient targets for incremental traffic investment.

---

## Phases

- [x] Phase 1: Project Charter (`reports/project_charter.md`)
- [x] Phase 2: Data Overview (`01_data_overview.ipynb`)
- [x] Phase 3: EDA (`02_eda.ipynb`)
- [x] Phase 4: Funnel Analysis (`03_funnel_analysis.ipynb`)
- [x] Phase 5: Product Analysis (`04_product_analysis.ipynb`)
- [ ] Phase 6: Channel & Budget Optimization (`05_budget_optimization.ipynb`)
- [ ] Phase 7: Experiment Design
- [ ] Phase 8: Dashboard
