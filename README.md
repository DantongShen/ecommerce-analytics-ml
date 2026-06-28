# End-to-End E-commerce Analytics and Revenue Modeling

An end-to-end data analysis project using the Google Analytics Sample Ecommerce Dataset on BigQuery. The goal is to understand user behavior across the purchase funnel, evaluate channel performance, and produce data-driven recommendations for budget allocation and conversion improvement.

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
│   ├── 01_data_overview.ipynb      # schema, distributions, data quality
│   ├── 02_eda.ipynb                # traffic, device, geography, purchase behavior, campaign analysis
│   ├── 03_funnel_analysis.ipynb    # purchase funnel drop-off by device, channel, country, visitor type, and time
│   ├── 04_product_analysis.ipynb   # product revenue, conversion rates, brand comparison, revenue per visit
│   ├── 05_budget_optimization.ipynb # channel efficiency, diminishing returns model, session allocation optimization
│   ├── 06_purchase_prediction.ipynb # logistic regression, random forest, and gradient boosting to predict session-level conversion
│   └── 07_experiment_design.ipynb  # A/B test design for mobile navigation redesign, with sample size calculation and decision rules
├── images/                          # saved chart outputs
├── sql/
│   ├── funnel_analysis.sql          # all BigQuery queries from 03_funnel_analysis.ipynb
│   └── product_analysis.sql         # all BigQuery queries from 04_product_analysis.ipynb
├── data/                            # intermediate CSV outputs
│   ├── product_category_map.csv        # canonical product-to-category mapping
│   ├── product_category_normalized.csv # normalized categories and brand tags
│   └── source_medium_breakdown.csv     # session and revenue breakdown by source/medium
├── credentials/                     # gitignored, BigQuery service account key
├── reports/
│   └── project_charter.md           # objectives, scope, success metrics, and phase plan
├── experiments/
└── dashboard/
```

---

## Traffic & Channel Insights

- **US concentration**: the US generates 94% of transactions from 40% of sessions. Canada is the only other market with meaningful traction. All other countries convert below 0.16%.
- **Referral is the highest-quality channel**: ~6.25% conversion rate and $718K revenue despite ranking 4th in sessions (after excluding `analytics.google.com`, a non-converting internal traffic source). Social drives reach but almost no revenue.
- **Engagement predicts purchase**: purchasers view a median of 37 pages vs 2 for non-purchasers. Below ~10 pageviews, almost no one buys.
- **Holiday traffic does not equal holiday revenue**: November had the most sessions but the lowest conversion rate. December had the highest conversion but one of the lowest AOVs - holiday buyers are decisive but spend less per order.
- **Campaign tracking is sparse**: only 3 of 8 named campaigns generated transactions, totalling less than 2% of total revenue. AW - Accessories and AW - Dynamic Search Ads are the only campaigns with measurable return.
- **AOV needs context**: Display, Japan, and April 2017 all appeared to be high-value segments, but each was driven by a small number of extreme orders rather than consistent performance.
- **Mobile barely converts**: Chrome at 1.76% vs Safari at 0.43%. Mobile browsers convert below 0.2%.

---

## Funnel Insights

- **86% of sessions never see a product page.** The biggest loss in the funnel is at discovery, not checkout. Only 1.30% of all sessions result in a purchase.
- **Checkout friction is at Payment and Review, not the confirm button.** Significant drop-off occurs entering payment details and at the review screen; almost no one abandons after reaching the final confirm step.
- **Mobile converts at 4x lower rate than desktop.** Drop-off starts from the first funnel step — mobile users are less likely to reach a product page, add to cart, and complete checkout at every stage. The gap between mobile and desktop widens further at Add to Cart.
- **Referral converts best because it enters the funnel deeper and drops off less.** Social drives high volume but 98% of sessions never click into a product.
- **International traffic faces a checkout wall.** US converts at 28x the rate of Rest of World. International users who do reach checkout drop off at 89% before completing - a signal of shipping restrictions or unsupported payment methods.
- **Returning visitors are 22% of sessions but 61% of purchases.** They convert at 6x the rate of new visitors, and their drop-off rate declines at every funnel step.
- **Best time to convert: Monday and Friday, 10 AM - 2 PM Pacific.** Weekday intent is nearly double weekend rates.

---

## Product Insights

- **Revenue is concentrated but not extreme:** the top 20% of products account for 69.4% of revenue; the bottom 50% contribute only 7.5%. A manageable set of ~50 products drives half of all revenue.
- **Apparel dominates revenue, Office dominates volume:** Apparel accounts for 39.3% of total revenue. Office leads in units sold at a low effective price, driven by bulk journal and notebook orders that signal corporate gifting.
- **Bags and Electronics have a view-to-cart problem (~18%):** both categories draw high traffic but rarely convert to add-to-cart. Accessories convert best into the cart (41%) but worst out of it (22%) - impulse adds that get abandoned if the session does not result in a primary purchase.
- **Google brand has a consistent purchase conversion advantage:** in matched-pair comparisons (same product, different brand), Google cart-to-purchase exceeds YouTube in all 4 pairs and Android in 3 of 4. Google converts at 2x YouTube's rate on journals even when priced higher, indicating strong brand-driven pricing power.
- **Two drivers of high revenue per detail-view session:** high unit price (premium items generate meaningful revenue even with modest purchase rates) and bulk buying (cheap items bought in large quantities per session, likely corporate gifting). The highest-traffic high-yield products are the most efficient targets for incremental traffic investment.

---

## Channel Optimization Insights

- **Social and Affiliates are confirmed candidates for reallocation:** Social drives 226K sessions but only $8K revenue (0.06% conversion); Affiliates yields $654 over the full year. Both hit the optimizer's 30% floor in every scenario — the most robust finding in the model.
- **Referral is the highest-efficiency controllable channel:** log revenue coefficient ~13x Paid Search. Most of its $718K revenue is last-non-direct-click attributed — users acquired via referral touchpoints who returned and purchased later. Growing Referral means more acquisition touchpoints, not same-session conversions.
- **Recommended starting point is S3 (Moderate rebalance):** halve Social and Affiliates, double Referral, grow Paid Search and Display 50%. Projects +$51K revenue vs baseline.
- **Dollar optimization is not possible with this dataset:** no ad spend data means the model optimizes session counts as a proxy for budget. The critical unknown is cost-per-session by channel — particularly whether Paid Search CPC is lower than revenue per Paid Search session. Without this, scenarios are directional only.

---

## Purchase Prediction Insights

- **All three models reach AUC 0.98+**: Logistic Regression (0.9828), Random Forest (0.9860), Gradient Boosting (0.9863). The ~0.004 gap confirms the signal is largely linear — LR captures most of it already.
- **Top three predictors across all models**: `is_us`, `pageviews`, `is_returning`. Geography and engagement depth dominate; channel and time features are secondary once visitor behavior is known.
- **`ch_Referral` coefficient is near zero after controlling for `is_returning`**: validates the notebook 05 attribution finding — Referral's 6.25% raw conversion rate is returning visitor behavior, not a channel effect.
- **`ch_Affiliates` is the strongest negative predictor**: worst channel even after controlling for all session characteristics; consistent with the budget optimization recommendation to cut it.
- **Feature ranking shifts between model types**: LR ranks `is_us` first (largest log-odds effect for a binary variable); tree models rank `pageviews` first (continuous variable exploitable across many splits). RF spreads importance across `pageviews` + `time_on_site` (77% combined); GB concentrates ~85% on `pageviews` alone.
- **Low precision (0.18–0.21) is a base rate problem, not a model problem**: with 1.30% conversion rate, even a well-calibrated model flags many non-buyers. Use the raw probability score for ranking sessions rather than a binary prediction.

---

## Experiment Design Insights

- **Target: mobile session to product page, not checkout.** The 86% first-step drop-off is the biggest funnel loss; mobile is worse at 12.38% product page view rate vs 14.45% on desktop.
- **Intervention: mobile homepage navigation redesign** — simpler menu structure, more prominent category entry points. Tablet excluded (smallest segment, screen size closer to desktop).
- **Primary metric: mobile product page view rate.** Directly measures whether the change moves users into the funnel. Guardrail metrics are desktop conversion rate and desktop revenue per session, which should be unchanged by a mobile-only intervention.
- **At 20% MDE, only 2,372 sessions per group are needed** (4,744 total). At 570 avg daily mobile sessions, raw duration is 8.3 days, rounded up to 2 weeks to cover a full weekday/weekend cycle.
- **Secondary metric is revenue per session, not add-to-cart or conversion rate.** Downstream UX is unchanged by the navigation intervention; add-to-cart and conversion rates for users who already reach a product page should not be expected to move.

---

## Phases

- [x] Phase 1: Project Charter (`reports/project_charter.md`)
- [x] Phase 2: Data Overview (`01_data_overview.ipynb`)
- [x] Phase 3: EDA (`02_eda.ipynb`)
- [x] Phase 4: Funnel Analysis (`03_funnel_analysis.ipynb`)
- [x] Phase 5: Product Analysis (`04_product_analysis.ipynb`)
- [x] Phase 6: Channel & Budget Optimization (`05_budget_optimization.ipynb`)
- [x] Phase 7: Purchase Prediction (`06_purchase_prediction.ipynb`)
- [x] Phase 8: Experiment Design (`07_experiment_design.ipynb`)
- [ ] Phase 9: Dashboard
