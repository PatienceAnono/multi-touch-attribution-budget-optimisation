# Authoritative metrics and definitions

These are the official dataset figures for this project. The notebook calculates them in **Section 5A** and saves them to `data/processed/dataset_metrics.csv` on every run. The README, the stakeholder report and any dashboard should quote these numbers.

They are calculated on the **cleaned** dataset, after the 99 duplicate copies created by double-fired tracking pixels are removed. That is the data every attribution model uses.

## Figures

| Metric | Raw (before cleaning) | **Cleaned (authoritative)** |
|---|---:|---:|
| Touchpoints | 11,292 | **11,193** |
| Customers | 3,500 | **3,500** |
| Customer journeys | 3,500 | **3,500** |
| Converted journeys | 1,454 | **1,454** |
| Conversion rate | 41.54% | **41.54%** |
| Revenue | $229,294.41 | **$229,294.41** |
| Paid spend | $20,086.85 | **$19,901.84** |
| Blended ROAS | 11.42x | **11.52x** |
| Multi-touch share | 84.3% | **84.2%** |

Data period: 1 January to 19 December 2024 (354 days, about 50.6 weeks).

Cleaning only removes the duplicate copies, so journeys, conversions and revenue are identical before and after.

## Definitions

| Metric | Definition |
|---|---|
| Touchpoints | Number of rows in the dataset |
| Customers | Distinct `customer_id` |
| Customer journeys | Distinct `journey_id`. There is exactly one journey per customer in this dataset |
| Converted journeys | Journeys where `journey_converted` = 1 |
| Conversion rate | Converted journeys ÷ customer journeys |
| Revenue | Sum of `journey_order_value`, counted **once per converted journey**. This is the total that Last Click, First Click, Time Decay and Shapley distribute across channels |
| Paid spend | Sum of `touchpoint_cost_usd` across the whole data period. This is a **full-period total, not a weekly or monthly figure**. Direct and SEO/Organic have zero cost |
| Blended ROAS | Revenue ÷ paid spend |
| Multi-touch share | Journeys with more than one touchpoint ÷ customer journeys |

## Where the older numbers came from

| Number quoted before | Where | What it actually was | Now |
|---|---|---|---|
| $231,469 revenue | v1 notebook Section 4, v1 report | Raw sum of `order_value_usd` at touchpoint level. It double-counts converting touchpoints that the pixel duplicated (1,467 converting rows for 1,454 journeys) | $229,294 |
| 3,495 journeys, 1,451 conversions, $228,903 revenue, $19,717 spend | Phase 1 clean-up | De-duplication dropped **both** rows of each duplicate pair, deleting 5 whole journeys (3 of them converted) | 3,500 · 1,454 · $229,294 · $19,902 |
| "1,454 of 3,495" | v1 README | A raw count mixed with an over-cleaned count | 1,454 of 3,500 |
| $20,087 spend, 11.5x ROAS | v1 notebook and report | Raw spend including duplicate fires | $19,902, 11.5x |
| "$6,487 per week on Influencer", "weekly budget" | v1 README, notebook, report | Full-period totals mislabelled as weekly | $6,549 for the 2024 period (after correct de-duplication) |
| "$20,000+ monthly budget" | v1 notebook Section 2 | Spend is for the whole of 2024 | About $19,900 of paid spend in 2024 |
| $4.8M per year cost of Last Click | v1 everywhere | Weekly ×52 error, Shapley used as its own judge, linear response, broken Shapley | +$5K to +$20K per year (data-driven vs Last Click budget, scenario range) |
