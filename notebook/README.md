# Attribution Model Comparison Notebook (v2)

*This notebook sits alongside the Power BI work described in the [main README](../README.md). It compares six attribution models and turns them into budget recommendations.*

When a customer touches five channels before buying, who gets the credit, and does the answer change where the budget should go?

It does. This project builds six attribution models on the same customer journeys, turns each one into a budget, and estimates what moving the budget is realistically worth, without letting any single model judge itself.

---

## The problem in one paragraph

Most businesses make budget decisions from Last Click attribution, the default in Google Ads, Meta Ads Manager and most analytics tools. Last Click gives 100% of the credit to the channel touched right before the purchase. Paid Search tends to sit at the end of journeys (people search when they are ready to buy), while Paid Social and Influencer tend to start them. Under Last Click, the closers look excellent and the openers look weak, and the budget follows.

In this dataset the business spent $6,549 on Influencer in 2024, 33% of paid media, for about 13% of revenue credit under every multi-touch model and the lowest return of any paid channel.

---

## What this project builds

Six attribution models, built from first principles on the same 3,500 journeys:

| Model | The rule | The bias |
|---|---|---|
| Last Click | 100% to the final touchpoint | Overvalues channels that close (Paid Search, Email, Direct) |
| First Click | 100% to the first touchpoint | Overvalues channels that open (Paid Social, Influencer) |
| Linear | Equal split across all touchpoints | Treats every touch the same |
| Time Decay | Exponential weight toward the purchase date (7-day half-life) | Undervalues early, brand-building touches |
| **Shapley** (data-driven) | Average marginal contribution, from game theory | Shares credit among channels in converted journeys only |
| **Markov Chain** (data-driven) | Conversions lost when a channel is removed from the journey graph | First-order: looks one step back |

---

## Results

**Data:** 11,292 raw touchpoints (11,193 after removing 99 duplicate pixel fires) · 3,500 customer journeys · 7 channels · 1 Jan to 19 Dec 2024

**Conversions:** 1,454 (41.5%) · **Revenue:** $229,294 · **Paid spend:** $19,902 for the whole period · **Blended ROAS:** 11.5x · **Multi-touch journeys:** 84%

Definitions and the reconciliation with older figures: [`docs/metrics_definitions.md`](../docs/metrics_definitions.md).

**Share of revenue credit:**

| Channel | Last Click | Shapley | Markov | Takeaway |
|---|---|---|---|---|
| Paid Search | 35.4% | 23.8% | 23.0% | Last Click over-credits it by ~12pp |
| Paid Social | 14.7% | 21.0% | 20.0% | Under-credited by Last Click |
| Email | 19.1% | 15.8% | 16.4% | Strong, but an owned channel, not paid media |
| Influencer | 7.7% | 13.3% | 12.6% | Under-credited, yet still over-funded at 33% of paid spend |
| SEO/Organic | 10.6% | 12.6% | 13.4% | No media cost |
| Display | 4.1% | 7.4% | 7.6% | Under-credited; very small budget |
| Direct | 8.5% | 6.0% | 6.8% | No media cost |

Shapley and Markov, two very different methods, agree within 1 percentage point on every channel.

**Budget (paid media, 2024 period):**

| Channel | Current | Data-driven target | Step 1 (20% of the way) |
|---|---|---|---|
| Paid Search | $8,528 | $7,126 | $8,248 |
| Influencer | $6,549 | $3,958 | $6,031 |
| Paid Social | $4,189 | $6,256 | $4,602 |
| Display | $368 | $2,294 | $753 |

**Dollar impact** (change in paid-channel revenue over the year vs today's budget; every budget scored under all 6 models with diminishing returns β = 0.3–0.7):

```
Step 1 (20% move toward the data-driven target)   +$2.0K to +$16.4K   (median +$6.7K)
Full data-driven target                            +$4.5K to +$65.0K   (median +$22.0K)
Data-driven target vs Last Click budget            +$5.2K to +$20.3K   (judged by Shapley and Markov)
```

These are scenario ranges, not forecasts. The recommendation is to take Step 1 and run holdout tests on Influencer and Display before going further.

> **Correction:** an earlier version of this project reported "$4.8M per year" lost to Last Click. That figure treated full-year spend as weekly spend (×52), used Shapley as its own judge, and relied on a Shapley model that gave Email, Influencer, Display and SEO zero credit. All three problems are fixed. See [`CHANGELOG.md`](../CHANGELOG.md).

---

## The data-driven models

**Shapley values.** Each channel gets its average marginal contribution to the value a group of channels creates. The value of a group S is the revenue from converted journeys that used only channels in S. With that definition, credits are never negative and add up exactly to revenue; in practice each sale is shared equally among the distinct channels in its journey.

**Markov chain removal effect.** All 3,500 journeys, including the 2,046 that did not buy, are turned into a map of how customers move between channels. Removing a channel and re-computing the conversion probability shows how much each channel matters. Paid Search has the largest removal effect (55%), then Paid Social (48%) and Email (39%).

Both measure association along the journey, not proven causation. Only experiments show true incrementality.

---

## Data quality

**Tracking pixel double-fires:** 198 rows are flagged by `is_duplicate_flag`. They are 99 pairs: the same session recorded twice. The tracking system flags **both** rows of a pair, so the notebook keeps the first row and drops the copy. (Dropping every flagged row, as an earlier version did, deleted 5 whole journeys.)

**Category typos (18 rows):** `channel_category` values like `'PAID'`, `'Earnd'`, `'ownd'` are re-derived from the `channel` column.

**Missing values:** small gaps in `campaign_name`, `device_type` and `touchpoint_hour` are filled. The 58% missing `conversion_date` is structural: those journeys did not convert.

---

## Notebook structure

| Sections | Content |
|---|---|
| 1–3 | Executive summary, business problem, setup |
| 4–5 | Data loading, quality assessment, cleaning pipeline with validation |
| 5A | Authoritative dataset metrics |
| 5B | Customer journey analysis |
| 6–10B | Six attribution models (Last Click, First Click, Linear, Time Decay, Shapley, Markov Chain) |
| 11 | Side-by-side model comparison |
| 12 | Budget reallocation under each model |
| 13 | Dollar impact: every budget scored under every model, with diminishing returns |
| 14 | Executive dashboard |
| 15 | Recommendations, roadmap and limitations |
| 16 | Export: clean data, model results, Power BI table, scenario grid |

---

## Files used and produced by this notebook

```
notebook/Attribution_Model_Comparison.ipynb   ← this notebook
data/raw/attribution_customer_journeys.csv    ← input (synthetic, 11,292 rows)
data/processed/                               ← outputs: dataset_metrics, results for all 6 models,
                                                 budget_reallocation, dollar_impact_summary / _scenarios
                                                 (attribution_clean.csv and attribution_powerbi.csv are
                                                 regenerated on each run and not committed)
visuals/1_ … 5_*.png                          ← charts and executive dashboard
reports/Attribution_Model_Stakeholder_Report.docx   ← stakeholder report v2
docs/metrics_definitions.md                   ← official metrics and reconciliation
CHANGELOG.md · requirements.txt               ← change history and dependencies
```

---

## How to run it

```bash
git clone https://github.com/PatienceAnono/multi-touch-attribution-budget-optimisation.git
cd multi-touch-attribution-budget-optimisation

pip install -r requirements.txt
jupyter notebook notebook/Attribution_Model_Comparison.ipynb
```

Run Kernel → Restart & Run All. The notebook finds the repository root itself, so it works when Jupyter is started from the repo root or a subfolder. All outputs go to `data/processed/` and `visuals/` at the repo root. The full run takes about 10–60 seconds depending on your machine; the Shapley step (127 channel combinations) and the Markov chain are the slowest parts.

**Requirements:** see `../requirements.txt` (pandas, numpy, matplotlib, seaborn, Jupyter). Tested on Python 3.13; originally developed on Python 3.9 (Anaconda).

No external attribution libraries. All six models are implemented from first principles so the code is readable and auditable.

---

## A few honest caveats

**Attribution is not incrementality.** Shapley and Markov split observed revenue among the channels customers touched. They cannot prove a channel caused a sale. Holdout or geo experiments are needed for that.

**The dollar impact rests on assumed response curves.** Big moves (Display growing about six times in the full target) are where those assumptions are weakest. That is why the plan starts with a 20% step.

**Email is held fixed.** It costs about $0.15 per send and its revenue depends on list size and engagement, not on buying more media, so it is managed as an owned channel rather than reallocated.

---

## About

**Patience Anono** — Data Analyst & Marketing Analytics Specialist

📧 hello@padataanalytics.com  
🌐 [padataanalytics.com](https://padataanalytics.com)  
💼 [LinkedIn](https://www.linkedin.com/in/patience-anono-22ab06176/)

---

*Dataset is synthetic, built to mirror real e-commerce attribution data structures. All analysis methodology and business logic is real and production-applicable.*
