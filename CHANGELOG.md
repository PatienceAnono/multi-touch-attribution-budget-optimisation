# Changelog

> Phases 1 and 2 were carried out in the local working folder, where the notebook sat at the repository root. In this GitHub repository it lives at `notebook/Attribution_Model_Comparison.ipynb`; it finds the repository root itself, so it runs unchanged. The Power BI work in `POWERBI/` and `reports/*.md` is separate and was not changed.


## Power BI project files and README alignment (21 November 2026)

- Added the full Power BI project (`POWERBI/…Report/` and `…SemanticModel/`) so the dashboard opens from the repository. The local cache (`cache.abf`) and local settings are excluded by `POWERBI/.gitignore`.
- The semantic model now reads its two CSVs through a `DataFolder` parameter instead of a fixed personal path. Set it once under **Transform data → Edit parameters**.
- README: clarified that the five models (including Position-Based, 3-day half-life) belong to the Power BI pipeline and the notebook uses six (7-day half-life, Shapley, Markov). Updated the reproduce steps, project structure and future enhancements.

## Phase 2: methodology fixes (21 November 2026)

This phase fixes the analytical problems found in Phase 1. **Results change in this phase.** The notebook runs end to end with no errors, and every attribution model distributes exactly the same total revenue ($229,294).

### 1. De-duplication (Phase 1 open issues 5 and 6)

- **Problem.** `is_duplicate_flag` is set on **both** rows of each double-fired session (99 sessions × 2 rows = 198 rows). The old cleaning dropped all 198 rows. That deleted 5 whole journeys (3 of them converted), left 10 converted journeys without a converting touchpoint, and left `total_touchpoints` wrong for 94 journeys. Because of that, the Linear model distributed $227,227 instead of the full revenue.
- **Fix.** Keep the first row of each flagged session and drop only the copy (99 rows). A new validation step (Step 8) checks that `total_touchpoints` matches the actual rows, that every converted journey keeps its converting touchpoint, and that no journey is lost.
- **Effect on the official figures.** 11,193 touchpoints · 3,500 journeys · 1,454 conversions (41.5%) · $229,294 revenue · $19,902 paid spend · 11.5x ROAS. Phase 1's 3,495 / 1,451 / $228,903 / $19,717 are superseded.

### 2. Shapley model rebuilt (open issue 2)

- **Problem.** v(S) was the conversion rate of all journeys that *contained* coalition S. That is not a valid characteristic function: marginal values were often negative and were then clipped to zero and rescaled. This gave Email, Influencer, Display and SEO exactly 0% and Direct 52%.
- **Fix.** v(S) = revenue from converted journeys whose channels are all inside S. v(∅) = 0 and v is monotone, so no credit is negative and credits sum exactly to revenue. The code asserts both properties and checks the result against the equivalent "equal split across a journey's distinct channels".

### 3. Markov chain model added

- First-order Markov chain over all 3,500 journey paths (converting and non-converting), with the removal effect per channel.
- It agrees with Shapley within 1 percentage point on every channel, an independent cross-check.

### 4. Weekly vs annual (open issue 1)

- All spend and revenue are totals for the data period (1 Jan to 19 Dec 2024). Every "weekly" label was removed and nothing is multiplied by 52. The notebook prints the period length (354 days) and the average weekly spend for reference.

### 5. Circular dollar impact replaced (open issue 3)

- **Problem.** Projected revenue = budget × Shapley ROAS, so Shapley always won, and revenue was assumed to grow in a straight line with spend.
- **Fix.** Every budget is scored under every model's view of channel performance, with diminishing returns (revenue ∝ spend^β, β = 0.3, 0.5, 0.7). The notebook shows the full matrix, the own-model bias, and worst / median / best results per budget.
- **Phased plan.** "Step 1" moves 20% of the way from today's budget to the data-driven target (the average of the Shapley and Markov budgets).
- **Email held fixed.** Email is an owned channel (about $0.15 per send), so only bought media (Paid Search, Paid Social, Influencer, Display) is reallocated. Without this, about 90% of every projected gain came from growing Email spend 15 times, which is not realistic.
- **New headline.** Step 1 is worth +$2.0K to +$16.4K paid revenue a year (median +$6.7K). Judged by the data-driven models, the data-driven target beats the Last Click budget by $5.2K to $20.3K a year. This replaces "$4.8M per year".

### 6. Narrative aligned with results (open issue 4)

- Executive summary, model table, Section 15 recommendations, roadmap and limitations were rewritten from the new results. Hard-coded "key insight" prints are now calculated from the data.
- The dashboard KPIs and recommendations are generated from the results. Dollar signs are escaped so matplotlib no longer renders them as maths text.
- "Full year 2024" is now "1 Jan to 19 Dec 2024" (open issue 7).

### 7. Outputs

| File | Change |
|---|---|
| `attribution_results_all_models.csv` | Adds Markov Chain columns |
| `budget_reallocation.csv` | Paid media only; adds Data-Driven Target and Step 1 columns |
| `dollar_impact_summary.csv` | New layout: worst / median / best change per budget, with the period |
| `dollar_impact_scenarios.csv` | **New**: the full grid (budget × model × β) |
| `attribution_powerbi.csv` | **Fixed**: it used to give every journey its last-touch channel's *total* revenue. It is now one row per journey × channel with each model's actual credit (sums verified against the model totals) |
| `visuals/4_dollar_impact.png`, `5_executive_dashboard.png` | Redesigned for the new method |
| `reports/Attribution_Model_Stakeholder_Report.docx` | Rewritten as v2 with the corrected results, both data-driven models, the new dollar-impact method and a "what changed from v1" section |
| `README.md`, `docs/metrics_definitions.md` | Updated to the corrected figures and method |

### Still open

- Verified end to end on Python 3.13 and on the original Python 3.9.12 / Anaconda environment (pandas 1.4.2), with identical results.
- The response curves are assumptions. Holdout tests on Influencer and Display would replace them with measured curves.
- Phase 1 and 2 work merged into the GitHub repository alongside the existing Power BI project (November 2026).

---

## Phase 1: repository clean-up (21 November 2026)

The goal of this phase was to turn the project folder into one clean, reproducible repository **without changing any attribution results or financial modelling**. All model outputs were checked after the clean-up and match the previous outputs exactly.

### Original project preserved

A full copy of the project as it was before any change is saved as `_backup/marketing-attribution-analysis_original_2026-11-21.zip`. The `_backup/` folder is listed in `.gitignore`, so it stays on your computer and is not published to GitHub.

### Notebook retained

`Attribution_Model_Comparison.ipynb` at the repository root was kept as the single canonical notebook.

- It was the most recently edited copy, and it ran without errors.
- Its code was identical to `Notebook/Attribution_Model_Comparison.ipynb` except for one blank line.
- The `Notebook/` copy could not run, because it looked for `data/raw/...` relative to the `Notebook/` folder and stopped at the data-loading cell.

### Files removed

These duplicates are retired. As of 21 November 2026 they still had to be deleted by hand from the local folder, because Claude's folder access can't delete files. None of them is needed to run the project.

| Removed | Why |
|---|---|
| `Notebook/Attribution_Model_Comparison.ipynb` | Duplicate notebook that could not run |
| `Notebook/.ipynb_checkpoints/` | Jupyter autosave files |
| `Notebook/data/processed/*.csv` (5 files) | Duplicate outputs. They now live in `data/processed/` and are regenerated by the notebook |
| `Notebook/visuals/*.png` (5 files) | Duplicate charts. They now live in `visuals/` |
| `.ipynb_checkpoints/` | Jupyter autosave files |
| `attribution_powerbi.csv` (root) | Older copy of `data/processed/attribution_powerbi.csv` |
| `dollar_impact_summary.csv` (root) | Older copy of `data/processed/dollar_impact_summary.csv` |

All of these are still inside the backup zip.

### Files moved or regenerated

| File | Change |
|---|---|
| `data/processed/*.csv` | Regenerated by the notebook. Before, this folder was empty and the outputs only existed in `Notebook/data/processed/` |
| `visuals/1_journey_analysis.png` | Regenerated. It was previously only in `Notebook/visuals/` |
| `visuals/2_…` to `visuals/5_…` | Regenerated from the canonical notebook |
| `reports/Attribution_Model_Stakeholder_Report.docx` | Same location. Dataset figures corrected (see below) |

No files were renamed.

### Files added

| File | Purpose |
|---|---|
| `requirements.txt` | Python dependencies |
| `.gitignore` | Keeps checkpoints, environments, large generated CSVs and the local backup out of Git |
| `CHANGELOG.md` | This file |
| `docs/metrics_definitions.md` | Authoritative metrics, their definitions and how they reconcile with older numbers |
| `data/processed/dataset_metrics.csv` | Raw vs cleaned metrics, written by the notebook on each run |

### Notebook changes

None of these changes touches the cleaning steps, the five attribution models, the budget reallocation or the dollar-impact calculation.

1. **Paths.** The notebook now finds the repository root itself (the folder containing `data/raw/attribution_customer_journeys.csv`) and reads and writes everything relative to it. It works when Jupyter is started from the repo root or from a subfolder.
2. **Section 4 overview.** It is now labelled as raw, pre-cleaning figures, with a pointer to Section 5A.
3. **New Section 5A: Authoritative Dataset Metrics.** This section shows a raw vs cleaned table with definitions and saves `data/processed/dataset_metrics.csv`.
4. **Header and Section 2 text.** The dataset line now reads 3,495 journeys and 11,094 cleaned touchpoints. The "$20,000+ monthly budget" wording was replaced with the actual 2024 paid spend.
5. **Inline chart size.** `plt.rcParams['figure.dpi'] = 72` keeps the notebook file small on newer matplotlib versions. Saved PNGs still use dpi=150.
6. **Cell IDs.** Cell IDs were added, as required by current Jupyter versions.

### Documentation changes

- **README:** corrected the dataset figures (3,495 journeys, 1,451 conversions, $228,903 revenue, $19,717 spend, 11.6x ROAS), updated the project structure, fixed the clone URL to `PatienceAnono/multi-touch-attribution-budget-optimisation`, and switched install instructions to `requirements.txt`.
- **Stakeholder report (.docx):** corrected the revenue ($231,469 → $228,903), the journey count (3,500 → 3,495), touchpoints (11,094 after cleaning), spend ($20,087 → $19,717) and blended ROAS (11.5x → 11.6x). Headline findings, the $4.8M figure and the "weekly" wording were left unchanged for Phase 2.

### Data published to GitHub

- `data/raw/attribution_customer_journeys.csv` (2.8 MB) is kept. The README states it is synthetic, and a scan found no names, emails or phone numbers. It is needed to reproduce the analysis.
- `attribution_clean.csv` (2.8 MB) and `attribution_powerbi.csv` (0.8 MB) are not committed. They are derived from the raw file and regenerated by the notebook.

### Verification

- Ran the canonical notebook from a clean kernel, from start to finish: 35 of 35 code cells ran with no errors (Python 3.13, pandas 3.1.1).
- Ran it again from a subfolder, and outputs went to the correct root folders.
- Compared every regenerated CSV with the previous outputs: identical, apart from floating-point rounding of about 1e-15.

### Open issues carried into Phase 2

These were found during Phase 1 and left unchanged at the time. **All seven were resolved in Phase 2 (see above).**

1. **Spend period mislabelled.** Spend is a full-period (2024) total but is presented as weekly spend, and gaps are multiplied by 52. The "$4.8M per year" headline is therefore about 52 times too high.
2. **Shapley coalition value.** v(S) is the conversion rate of journeys that *contain* S, and negative marginal values are clipped to zero. This gives Email, Influencer, Display and SEO exactly 0% and Direct 52%.
3. **Circular dollar impact.** Projected revenue uses Shapley ROAS as ground truth, so the Shapley budget wins by construction.
4. **Narrative contradicts results.** Section 15.2 says Email and Influencer should be increased or protected, but the Shapley results give both zero budget.
5. **De-duplication side effects.** Removing flagged rows deletes 5 journeys entirely (3 of them converted). It also leaves 10 converted journeys without a converting touchpoint, and leaves `total_touchpoints` stale for 94 journeys.
6. **Linear model.** Because `total_touchpoints` is stale, Linear distributes $227,227 instead of $228,903.
7. **Small labels.** The README calls the data "Full year 2024", but it ends on 19 December.
