# Multi-Touch Attribution Model Comparison & Budget Optimisation

This project answers what I think is the most underrated question in marketing analytics: when a customer interacts with five different channels before buying, who gets the credit — and does it actually matter which answer you choose?

It matters. A lot. This notebook quantifies exactly how much.

---

## The problem in one paragraph

Most businesses run their marketing budget decisions off Last Click attribution — the default in Google Ads, Facebook Ads Manager, and most analytics platforms. Last Click gives 100% of the conversion credit to whatever channel the customer touched right before buying. Which sounds reasonable until you notice that Paid Search almost always appears at the end of a journey (customers search when they're ready to buy) while Paid Social and Influencer almost always appear at the beginning (they start the journey). Under Last Click, Paid Search looks exceptional and everything that started the journey looks weak. The budget follows the story the data is telling. And the story is wrong.

In this dataset, the marketing team was spending $6,487 per week on Influencer — 32.9% of the total paid budget — while spending $265 per week on Email. Last Click ROAS for Influencer: 2.7x. Last Click ROAS for Email: 163x. That inversion should have been a red flag years before this analysis.

---

## What this project builds

Five attribution models, built from scratch on the same customer journey dataset, compared side by side with their implied budget allocations and the projected dollar impact of choosing between them.

| Model | The rule | The bias |
|---|---|---|
| Last Click | 100% to the final touchpoint | Overvalues Paid Search and Direct |
| First Click | 100% to the first touchpoint | Overvalues Paid Social and Influencer |
| Linear | Equal split across all touchpoints | Treats a bounce and a 10-minute session the same |
| Time Decay | Exponential weight toward purchase date | Undervalues brand-building activity |
| Shapley (Data-Driven) | Average marginal contribution via game theory | Computationally intensive; the gold standard |

---

## Results

**Dataset:** 11,292 touchpoints · 3,500 customers · 7 channels · Full year 2024

**Conversion rate:** 41.5% (1,454 converting journeys out of 3,495)

**84% of journeys are multi-touch** — meaning the question of who gets the credit is not academic. It affects the majority of revenue.

**How Last Click distorts channel credit:**

| Channel | Last Click | Shapley | Difference |
|---|---|---|---|
| Paid Search | 35.6% | 41.9% | Underfunded despite being strong |
| Paid Social | 15.0% | 6.2% | Overfunded — contribution lower than clicks suggest |
| Influencer | 7.6% | 0.0% | Massively overfunded at 32.9% of budget |
| Email | 18.9% | 0.0% | High Last Click ROAS driven by selection bias, not causation |

**Dollar impact:**

```
Shapley-optimised budget  →  $203,603 projected revenue per campaign  (MROI: 10.3x)
Last Click budget         →  $110,587 projected revenue per campaign  (MROI: 5.6x)
Weekly gap                →  $93,016
Annualised gap            →  $4,836,832
```

To be clear about what that $4.8M figure means: it is the projected revenue difference between running the budget according to Last Click's recommendations versus Shapley's recommendations, using Shapley ROAS as the ground truth for true channel performance. It assumes 52 campaign cycles per year at the current spend level. It also assumes channel ROAS stays constant as spend changes — which it won't. The number is directional, not a guarantee. The point is that model choice has a real financial consequence, not just a methodological one.

---

## The Shapley model — why it's the right one

Shapley values come from cooperative game theory. Lloyd Shapley won the Nobel Prize for this work in 2012. The idea: each channel's credit equals its average marginal contribution to conversion probability across every possible combination of channels it could appear in.

In practice this means: if adding Email to a journey that already has Paid Search and Paid Social consistently improves conversion probability — across many different such journeys — Email gets meaningful credit. If adding Email rarely changes conversion probability (maybe because high-intent customers are already on the email list regardless of whether they would have converted), Email gets low credit.

This is why Email scores zero in this dataset. It correlates strongly with conversion (163x Last Click ROAS) but does not independently cause it. Shapley separates correlation from causation in a way that Last Click simply cannot.

Google Analytics 4's "Data-Driven Attribution" is Shapley. Meta's Robyn marketing mix modelling framework is built on similar principles. This is where the industry has been moving for the last several years, even if most businesses are still making budget decisions off Last Click exports from their ad platforms.

---

## Data quality

The raw dataset had five issues. Two worth explaining:

**Tracking pixel double-fires (198 rows):** The `is_duplicate_flag` column was generated by the tracking system and identified 198 rows where the same session triggered the pixel twice within milliseconds. These are not full-row duplicates — they have unique `touchpoint_id` values — but they share a `session_id` and represent the same user action being recorded twice. Removing them before attribution prevents double-counting channel touchpoints.

**Category typos (18 rows):** `channel_category` had values like `'PAID'`, `'Earnd'`, `'ownd'`, `'Payd'`. Rather than pattern-matching against the typos (which breaks on any new variant), these were re-derived from the `channel` column using a clean lookup. The result is consistent regardless of what the original entry said.

The 58.4% missingness in `conversion_date` and `days_to_conversion` is structural, not a data quality problem. Non-converting journeys have no conversion date by definition. The identical missing percentage across both columns confirms they are the same population.

---

## Notebook structure

The notebook runs in 16 sections:

| Sections | Content |
|---|---|
| 1–3 | Executive summary, business problem, setup |
| 4–5 | Data loading, quality assessment, cleaning pipeline |
| 5B | Customer journey analysis — path lengths, first/last touch by channel |
| 6–10 | The five attribution models built with documented functions |
| 11 | Side-by-side model comparison — revenue share table and heatmap |
| 12 | Budget reallocation under each model |
| 13 | Dollar impact analysis — projected revenue, weekly and annual gaps |
| 14 | Executive dashboard |
| 15 | Strategic recommendations and implementation roadmap |
| 16 | Export — clean CSV, all model results, Power BI dataset |

---

## Project structure

```
multi-touch-attribution-budget-optimisation/
│
├── Attribution_Model_Comparison.ipynb     ← Main notebook (16 sections)
│
├── data/
│   ├── raw/
│   │   └── attribution_customer_journeys.csv   ← Raw dataset (11,292 rows)
│   └── processed/
│       ├── attribution_clean.csv               ← After quality treatment
│       ├── attribution_results_all_models.csv  ← Revenue + ROAS, all 5 models
│       ├── budget_reallocation.csv             ← Current vs recommended spend
│       ├── attribution_powerbi.csv             ← Journey-level, Power BI ready
│       └── dollar_impact_summary.csv           ← Weekly + annual projections
│
├── visuals/
│   ├── 1_journey_analysis.png                 ← Path length, first/last touch
│   ├── 2_model_comparison.png                 ← Side-by-side + heatmap + rank
│   ├── 3_budget_reallocation.png              ← Stacked budget bars + change heatmap
│   ├── 4_dollar_impact.png                    ← Projected revenue + annual gaps
│   └── 5_executive_dashboard.png             ← One-page summary
│
├── Attribution_Model_Stakeholder_Report.docx  ← Full stakeholder write-up
│
└── README.md
```

---

## How to run it

```bash
git clone https://github.com/anonopatience/multi-touch-attribution.git
cd multi-touch-attribution

pip install pandas numpy matplotlib seaborn
jupyter notebook Attribution_Model_Comparison.ipynb
```

Run Kernel → Restart & Run All. All output directories are created automatically. The Shapley computation takes roughly 30–60 seconds depending on your machine — it is doing combinatorial maths across seven channels for every customer journey.

**Requirements:**
```
pandas, numpy, matplotlib, seaborn, itertools (built-in), math (built-in)
```

No external attribution libraries. All five models are implemented from first principles so the code is readable and auditable.

---

## A few honest caveats

**The Shapley model in this notebook is data-driven but simplified.** It uses actual journey channel combinations from the dataset to estimate coalition values — which is more defensible than theoretical assumptions — but a production Shapley implementation would use larger holdout experiments and more sophisticated statistical approaches to estimate marginal contribution.

**The $4.8M annual figure should be treated carefully.** It assumes ROAS stays constant as spend changes, which it will not. Every channel has a saturation curve. The figure is meaningful as a directional estimate of the opportunity from better attribution. It is not a guaranteed outcome from implementing the recommended budgets.

**Email's zero Shapley credit does not mean Email does not work.** It means Email does not independently cause conversion in this dataset — likely because the customers on the email list are already high-intent. Email is still the right tool for customer retention and re-engagement. It just should not be measured or budgeted as a customer acquisition channel.

---

## About

**Patience Anono** — Data Analyst & Marketing Analytics Specialist

📧 hello@padataanalytics.com  
🌐 [padataanalytics.com](https://padataanalytics.com)  
💼 [LinkedIn](https://www.linkedin.com/in/patience-anono-22ab06176/)

---

*Dataset is synthetic, built to mirror real e-commerce attribution data structures. All analysis methodology and business logic is real and production-applicable.*
