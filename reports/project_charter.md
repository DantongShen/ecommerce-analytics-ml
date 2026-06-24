# Project Charter: End-to-End E-commerce Analytics and Revenue Modeling

## Background

The Google Merchandise Store (store.google.com) sells branded physical merchandise online. One year of its Google Analytics data (Aug 2016 - Aug 2017, ~900K sessions) has been made public for analysis. This project uses that dataset to evaluate channel and campaign performance, identify funnel drop-off points, and produce data-driven budget allocation recommendations.

---

## Business Objectives

1. **Identify which channels and campaigns drive real revenue** - not just traffic volume.
2. **Diagnose where users drop off in the purchase funnel** - to find the highest-leverage UX fix.
3. **Recommend a channel session reallocation** - modeling which controllable channels should receive more or less traffic, as a directional proxy for budget allocation. Actual dollar-level optimization is out of scope because spend data is not available in this dataset.
4. **Quantify the mobile conversion gap** - and assess whether it is an audience problem or a UX problem.

---

## Key Business Questions

From the EDA, these are the open questions the remaining phases will answer:

- Where exactly in the funnel do high-pageview users abandon? (Phase 4)
- What is the conversion rate at each step: product view → add to cart → checkout → purchase?
- Are Affiliates and dormant AW campaigns worth keeping at any budget level? (Phase 5)
- If Referral converts at ~6.25% and Social at 0.06%, what is the revenue impact of shifting Social budget to Referral/Paid Search? (Phase 6)
- Does the mobile conversion gap (Safari 0.43%, Android Webview 0.08%) reflect audience intent or a broken mobile experience? (Answered in Phase 4: gap opens at Add to Cart and holds through checkout - a UX friction problem, not an audience problem.)

---

## Success Metrics

| Metric | Current Baseline | Target |
|---|---|---|
| Overall conversion rate | 1.34% | Identify channels where lift is possible |
| Referral conversion rate | ~6.25% | Maintain or grow with higher session volume |
| Funnel drop-off at add-to-cart | Unknown | Quantify in Phase 4 |
| Revenue from named campaigns | <2% of total ($28K / $1.78M) | Dataset limitation: campaign ID, ad group, and creative fields are obfuscated in the demo dataset; attribution improvement is not actionable here |
| Mobile conversion rate | <0.2% | Diagnose root cause |

---

## Scope

**In scope:**
- Traffic, channel, and campaign performance analysis
- Purchase funnel analysis using hit-level data (`eCommerceAction.action_type`)
- Budget reallocation modeling (constrained optimization)
- A/B test design for one high-impact intervention

**Out of scope:**
- Real-time data or live dashboard updates
- Paid media cost data (not available in public dataset - ROI estimates will use proxy assumptions)
- Non-US market strategy (US = 94% of transactions; international analysis is descriptive only)

---

## EDA Key Findings (input to phases 4-8)

- **US concentration**: 40% of sessions, 94% of transactions. Canada is the only other converting market.
- **Channel quality gap**: Referral converts at ~6.25%, Social at 0.06%. Social drives 226K sessions but only $8K revenue.
- **Affiliates failed**: 16K sessions, 9 transactions, $654 revenue over the full year.
- **Display outliers**: dfa/cpm accounts for extreme AOV spikes ($47K single session). Display's $857 AOV is not a real signal.
- **Engagement signal**: purchasers view a median of 37 pages vs 2 for non-purchasers. Below ~10 pages, almost no one buys.
- **Campaign tracking gap**: only 3 of 8 campaigns tracked transactions, covering <2% of total revenue.
- **Mobile gap**: Chrome converts at 1.76%, Safari at 0.43%, mobile browsers below 0.2%.
