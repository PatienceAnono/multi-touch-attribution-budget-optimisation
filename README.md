# Multi-Touch Attribution & Marketing Budget Optimisation

> An end-to-end marketing analytics project combining Python, multi-touch attribution modeling, data validation, Power BI, DAX, and stakeholder-focused dashboard design.

**Built by PA Data Analytics**

---

## 📊 Project Overview

Marketing teams often struggle to answer:

> **Which marketing channels are contributing to conversions, and how should marketing investment be evaluated across the customer journey?**

This project builds an end-to-end **Multi-Touch Attribution & Marketing Budget Optimisation** workflow, starting with raw customer-journey data and progressing through data cleaning, validation, attribution modeling, reconciliation, Power BI semantic modeling, DAX, and an executive-ready dashboard.

### End-to-end workflow

```text
Raw Data
   ↓
Data Cleaning
   ↓
Data Quality Validation
   ↓
Customer Journey Validation
   ↓
Attribution Modeling
   ↓
Revenue Reconciliation
   ↓
Power BI Semantic Model
   ↓
DAX Measures
   ↓
Interactive Dashboard
   ↓
Stakeholder Insights
```

---

# 🎯 Business Objectives

The project was designed to answer five core business questions:

1. **How much conversion revenue is attributed to each marketing channel?**
2. **How does channel contribution change across different attribution models?**
3. **Which channels and campaigns appear efficient relative to marketing spend?**
4. **What does the customer journey look like before conversion?**
5. **How should attribution insights be interpreted when evaluating marketing investment?**

---

# 📁 Dataset

The project uses customer-level marketing journey data containing multiple touchpoints leading to either conversion or non-conversion.

The dataset contains information relating to:

- Customer journeys
- Touchpoint sequences
- Marketing channels
- Channel categories
- Campaigns
- Customer segments
- Regions
- Conversion status
- Order revenue
- Marketing spend
- Touchpoint timing

## Validated Dataset Summary

| Metric | Value |
|---|---:|
| Total Journeys | **3,500** |
| Converted Journeys | **1,454** |
| Touchpoints | **11,193** |
| Marketing Channels | **7** |
| Campaigns | **32** |
| Customer Segments | **5** |
| Regions | **5** |
| Conversion Revenue | **$229,294.41** |
| Marketing Spend | **$19,901.84** |
| Conversion Rate | **41.5%** |

The cleaned journey dataset retains the full journey population, including non-converted journeys.

The attribution output is restricted to converted journeys because attribution requires an observed conversion and associated revenue.

---

# 🧹 Data Cleaning & Quality Assurance

The raw dataset was audited before attribution modeling.

The objective was not simply to remove "bad" records, but to determine whether potentially problematic records could affect journey integrity, conversion attribution, or revenue reconciliation.

## Duplicate Investigation

The raw dataset contained:

- **99 duplicate journey-position groups**
- **198 duplicate rows**
- Every duplicate group contained exactly two records
- Both records were substantively identical
- Differences were limited to tracking-level fields such as `touchpoint_id`
- Both records were flagged as duplicates
- No conflicting duplicate groups were identified

## Canonical Deduplication Rule

Because every duplicate pair represented the same substantive touchpoint, the cleaning pipeline applied a deterministic rule:

> **Retain the lowest `touchpoint_id` for each `journey_id + touchpoint_position` duplicate group.**

This removed **99 duplicate rows**, rather than incorrectly removing both records from each duplicate group.

## Cleaning Validation

| Validation Check | Result |
|---|---:|
| Journeys preserved | **3,500 / 3,500** |
| Complete touchpoint sequences | **100%** |
| Converted journeys protected | **1,454 / 1,454** |
| Revenue before cleaning | **$229,294.41** |
| Revenue after cleaning | **$229,294.41** |
| Revenue difference | **$0.00** |

The final cleaning process therefore preserved the commercial integrity of the source dataset.

---

# 🏷️ Channel Classification

Channel categories were re-derived using the project's established channel-to-category mapping rather than relying on inconsistent source labels.

| Channel | Category |
|---|---|
| Paid Search | Paid |
| Paid Social | Paid |
| Display | Paid |
| Influencer | Paid |
| Email | Owned |
| SEO / Organic | Earned |
| Direct | Earned |

The cleaning process corrected **18 inconsistent channel-category records**.

---

# 📈 Attribution Modeling

Five attribution models were implemented from scratch at the touchpoint level using the validated cleaned dataset.

The models were applied to the **1,454 converted journeys**:

1. First Touch
2. Last Touch
3. Linear
4. Time Decay
5. Position Based

Each model allocates the same underlying conversion revenue differently according to its methodology.

## 1️⃣ First-Touch Attribution

First Touch assigns **100% of conversion credit to the first touchpoint**.

Useful for understanding:

- Customer acquisition
- Discovery
- Initial awareness
- Top-of-funnel channel contribution

It answers:

> **Which channel introduced the customer to the journey?**

## 2️⃣ Last-Touch Attribution

Last Touch assigns **100% of conversion credit to the final touchpoint before conversion**.

Useful for understanding:

- Closing interactions
- Lower-funnel activity
- Conversion-stage channels

It answers:

> **Which channel was present immediately before conversion?**

## 3️⃣ Linear Attribution

Linear attribution distributes conversion revenue equally across all touchpoints.

For a four-touch journey:

```text
25% / 25% / 25% / 25%
```

It provides a neutral multi-touch baseline where each recorded interaction receives equal credit.

## 4️⃣ Time-Decay Attribution

Time Decay gives greater weight to interactions closer to conversion.

The project uses a **3-day half-life**, derived from the dataset's observed median conversion window of approximately six days.

This is an explicit analytical assumption, not a fitted causal parameter.

## 5️⃣ Position-Based Attribution

The Position-Based model uses:

- **40%** → First touch
- **20%** → Middle touchpoints collectively
- **40%** → Last touch

Special handling was applied to short journeys:

- One-touch journey → **100%**
- Two-touch journey → **50% / 50%**

---

# ✅ Attribution Reconciliation

All five attribution models reconcile exactly to the source conversion revenue.

| Attribution Model | Total Attributed Revenue |
|---|---:|
| First Touch | **$229,294.41** |
| Last Touch | **$229,294.41** |
| Linear | **$229,294.41** |
| Time Decay | **$229,294.41** |
| Position Based | **$229,294.41** |

**1,454 / 1,454 converted journeys passed validation for every attribution model.**

The models were checked for:

- Weight sum = 1
- Attributed revenue = journey revenue
- Negative attribution weights
- Over-attribution
- Post-conversion touchpoints
- Duplicate attribution records

### Final result

```text
Revenue Difference = $0.00
```

across all five attribution models.

---

# 🔍 Key Attribution Findings

| Channel | First Touch | Last Touch | Interpretation |
|---|---:|---:|---|
| Paid Search | 14.2% | **35.4%** | Strong closing-stage role |
| Paid Social | **27.8%** | 14.7% | Stronger discovery role |
| Influencer | **19.2%** | 7.7% | Stronger top-of-funnel role |
| Email | 9.1% | **19.1%** | Stronger nurture / closing role |

### Paid Search

Paid Search increases from **14.2% → 35.4%** from First Touch to Last Touch, indicating a strong observed lower-funnel/closing role.

### Paid Social

Paid Social decreases from **27.8% → 14.7%**, indicating stronger observed contribution earlier in the customer journey.

### Influencer

Influencer decreases from **19.2% → 7.7%**, indicating stronger observed top-of-funnel/discovery contribution.

### Email

Email increases from **9.1% → 19.1%**, indicating a stronger observed nurture/closing role.

## 💡 Main Attribution Insight

> **There is no single attribution perspective that tells the complete marketing story.**

A channel can appear weak under one methodology and significantly more important under another. Stakeholders should therefore compare attribution models rather than treating one model as absolute truth.

---

# 🚦 Direct Traffic Analysis

Direct traffic was investigated before being included in the primary analysis.

| Position | Share |
|---|---:|
| First Touch | **27.6%** |
| Middle Touch | **37.3%** |
| Last Touch | **39.2%** |

Direct traffic was not overwhelmingly concentrated in one journey position.

Therefore:

> **Direct was retained as a genuine channel in the primary perspective.**

A separate Direct-excluded perspective was also created for comparison. Fully Direct journeys were excluded from that perspective rather than artificially transferring their revenue to another channel.

---

# 💰 Marketing Investment

Validated marketing spend:

| Channel | Marketing Spend |
|---|---:|
| Paid Search | **$8,528.47** |
| Influencer | **$6,549.29** |
| Paid Social | **$4,188.60** |
| Display | **$367.82** |
| Email | **$267.66** |
| Direct / SEO | **$0.00** |
| **Total** | **$19,901.84** |

Where a channel has zero recorded spend, ROAS is intentionally returned as blank rather than fabricating an efficiency value.

---

# 📊 Power BI Dashboard

The analytical outputs were transformed into a stakeholder-facing five-page Power BI dashboard.

The dashboard is designed for:

- Marketing managers
- Growth teams
- Commercial stakeholders
- Business leaders
- Analytics teams

The dashboard focuses on business questions rather than simply displaying metrics.

## 1️⃣ Executive Overview

### Purpose

Provide leadership with a concise overview of marketing performance.

### Includes

- Total Conversion Revenue
- Total Marketing Spend
- ROAS
- Conversion Rate
- Converted Journeys
- Distinct Customers
- Monthly attributed revenue and spend
- Channel attributed revenue ranking
- Executive insight callout

### Business Question

> **How is marketing performing overall, and where is attributed value concentrated?**

## 2️⃣ Attribution Analysis

### Purpose

The primary analytical page.

### Includes

- First Touch vs Last Touch comparison
- First Touch revenue
- Last Touch revenue
- Linear revenue
- Time Decay revenue
- Position Based revenue
- Channel attribution matrix
- Attribution revenue comparison
- Model comparison
- Insight callouts

### Business Question

> **How does the channel story change depending on the attribution methodology?**

## 3️⃣ Channel & Campaign Performance

### Purpose

Evaluate channel and campaign efficiency.

### Includes

- Marketing Spend vs Attributed Revenue
- Channel ROAS
- Campaign ranking
- Efficiency matrix
- Cost per conversion
- Channel filtering
- Campaign filtering

### Business Question

> **Which channels and campaigns appear most efficient relative to marketing spend?**

## 4️⃣ Customer Journey

### Purpose

Understand the structure of customer journeys before conversion.

### Includes

- Touchpoints per journey
- Days to conversion
- First-touch vs last-touch channel comparison
- Average touchpoints
- Average days to conversion
- Revenue per converted customer
- Customer segment filtering
- Regional filtering

### Business Question

> **What does the customer journey look like before conversion?**

## 5️⃣ Marketing Investment

### Purpose

Connect marketing investment with attributed value.

### Includes

- Marketing Spend vs Attributed Revenue
- ROAS across attribution models
- Cost per conversion
- Channel efficiency
- Investment interpretation
- Attribution-model comparison

### Business Question

> **Where is marketing investment producing attributed value, and how sensitive is that conclusion to attribution methodology?**

---

# 🏗️ Power BI Architecture

The project deliberately separates analytical computation from reporting.

## Python

Python was used for:

- Data cleaning
- Duplicate investigation
- Journey validation
- Attribution calculations
- Revenue reconciliation
- Cost and ROI analysis
- Analytical validation

## Power BI

Power BI was used for:

- Semantic modeling
- Date dimension
- Channel dimension
- DAX measures
- Interactive filtering
- Data visualization
- Executive reporting

The five attribution models were calculated in Python and consumed by Power BI.

```text
                    RAW DATA
                       │
                       ▼
                DATA CLEANING
                       │
                       ▼
               DATA VALIDATION
                       │
                       ▼
             ATTRIBUTION MODELS
                       │
                       ▼
                RECONCILIATION
                       │
                       ▼
              POWER BI MODEL
                       │
                       ▼
                     DAX
                       │
                       ▼
             INTERACTIVE REPORT
                       │
                       ▼
            STAKEHOLDER INSIGHTS
```

---

# 🧮 DAX & Semantic Modeling

The Power BI semantic model includes measures covering:

- Core business metrics
- Attribution models
- Model comparison
- Channel performance
- Campaign performance
- Customer analysis
- Time analysis
- QA and reconciliation

The attribution measures aggregate the validated Python attribution outputs rather than silently recreating the attribution methodology inside DAX.

This creates a clear separation between:

> **Analytical computation**

and

> **Business intelligence reporting**

---

# 🔎 Power BI Quality Assurance

The final Power BI report was reloaded under a fresh Power BI Desktop session and validated against the Python source of truth.

| QA Check | Result |
|---|---|
| Conversion Revenue | **$229,294.41** ✅ |
| First Touch Total | **$229,294.41** ✅ |
| Last Touch Total | **$229,294.41** ✅ |
| Linear Total | **$229,294.41** ✅ |
| Time Decay Total | **$229,294.41** ✅ |
| Position Based Total | **$229,294.41** ✅ |
| Marketing Spend | **$19,901.84** ✅ |
| Total Journeys | **3,500** ✅ |
| Converted Journeys | **1,454** ✅ |
| Channel Spend | **Matches Python** ✅ |
| Channel ROAS | **Matches Python** ✅ |

Visual QA checked:

- KPI precision
- Chart visibility
- Category clipping
- Page hierarchy
- Dashboard structure
- Branding
- Terminology
- Stakeholder readability

---

# 🎛️ Dashboard Interactivity

The Power BI report includes:

- Report-wide date filtering
- Channel filtering
- Campaign filtering
- Customer segment filtering
- Regional filtering
- Cross-filtering between visuals
- Native tooltips
- Consistent page structure
- Stakeholder-focused visual hierarchy

The report was designed to allow users to move from:

**Executive Summary → Attribution Analysis → Channel/Campaign Detail → Customer Journey → Investment Analysis**

---

# 📂 Project Structure

```text
multi-touch-attribution-budget-optimisation/
│
├── data/
│   ├── raw/
│   └── processed/
│
├── notebook/
│   └── Attribution_Model_Comparison.ipynb
│
├── visuals/
│   └── dashboard screenshots
│
├── POWERBI/
│   └── Marketing_Attribution_Dashboard.pbip
│
├── reports/
│   ├── DATA_CLEANING_REPORT_v2.md
│   ├── POWERBI_MODEL_DOCUMENTATION.md
│   ├── POWERBI_VISUAL_DESIGN_SPEC.md
│   └── POWERBI_FINAL_QA_REPORT.md
│
└── README.md
```

---

# 📚 Project Documentation

Supporting documentation covers the major stages of the project.

### Data Cleaning Report

Documents:

- Duplicate investigation
- Cleaning decisions
- Data-quality findings
- Journey preservation
- Revenue integrity
- Validation results

### Power BI Model Documentation

Documents:

- Semantic model
- Tables
- Relationships
- DAX measures
- Attribution integration
- QA checks

### Visual Design Specification

Documents:

- Page objectives
- Visual selection
- Stakeholder questions
- Layout decisions
- Interaction strategy
- Design system

### Final Power BI QA Report

Documents:

- Final report validation
- Numerical reconciliation
- Visual QA
- Dashboard readiness
- Known limitations

---

# ▶️ Reproducing the Analysis

Clone the repository:

```bash
git clone https://github.com/PatienceAnono/multi-touch-attribution-budget-optimisation.git
```

Navigate into the project:

```bash
cd multi-touch-attribution-budget-optimisation
```

Install the Python dependencies:

```bash
pip install pandas numpy matplotlib seaborn jupyter
```

Launch Jupyter:

```bash
jupyter notebook
```

Open:

```text
notebook/Attribution_Model_Comparison.ipynb
```

Run the notebook from the beginning to reproduce the analytical workflow.

---

# ⚠️ Limitations & Responsible Interpretation

## Attribution Is Not Causality

A channel receiving 30% of attributed revenue does **not** mean that the channel caused 30% of revenue.

Attribution distributes observed conversion revenue according to a defined methodology.

Therefore, this project uses terms such as:

- Attributed Revenue
- Attribution Share
- Modeled Contribution

rather than claiming:

- Incremental Revenue
- Causal Impact
- Guaranteed ROI

## Time-Decay Is Assumption-Based

The 3-day half-life was derived from the observed conversion window.

It is an analytical assumption and has not been fitted using a causal experiment.

## Attribution Models Are Perspectives

First Touch, Last Touch, Linear, Time Decay, and Position Based each make different assumptions about how credit should be distributed.

No single model should automatically be treated as the definitive representation of marketing impact.

## Budget Decisions Require Additional Evidence

Attribution should support — not replace:

- Incrementality testing
- Controlled experiments
- Holdout tests
- Marketing Mix Modeling
- Causal analysis

Increasing spend in a channel based solely on attributed revenue may not produce proportional incremental revenue.

## Missing Data

`device_type` and `touchpoint_hour` contain missing values.

These fields were documented rather than artificially imputed because they are not required by the five implemented attribution models.

---

# 💡 Key Business Recommendation

The central recommendation from this project is:

> **Do not make major marketing budget decisions using Last-Touch attribution alone.**

Instead, stakeholders should:

1. Compare multiple attribution perspectives.
2. Evaluate attributed revenue alongside actual marketing spend.
3. Examine channel and campaign efficiency.
4. Understand the customer's full journey.
5. Investigate how attribution methodology changes the investment story.
6. Use experiments and incremental evidence before making major budget reallocations.

The purpose of multi-touch attribution is therefore not simply to identify a "winning channel."

It is to help stakeholders understand:

> **How different attribution assumptions change the perceived contribution of each marketing channel.**

---

# 🎓 Skills Demonstrated

## Marketing Analytics

- Multi-touch attribution
- Customer journey analysis
- Channel performance analysis
- Campaign performance analysis
- ROAS analysis
- Marketing investment analysis
- Customer conversion analysis

## Data Analytics

- Data cleaning
- Data-quality investigation
- Duplicate resolution
- Journey validation
- Revenue reconciliation
- Exploratory data analysis
- Analytical documentation

## Business Intelligence

- Microsoft Power BI
- DAX
- Power Query
- Semantic modeling
- Date modeling
- Interactive dashboard development
- Executive dashboard design
- Data storytelling

## Technical

- Python
- Pandas
- NumPy
- Matplotlib
- Seaborn
- Jupyter Notebook
- Git
- GitHub

## Professional Analytics Skills

- Business problem definition
- Analytical reasoning
- Stakeholder communication
- Data storytelling
- Quality assurance
- Reproducible analytics
- Documentation
- Translating analysis into business recommendations

---

# 📌 Portfolio Positioning

### Project

**Multi-Touch Attribution & Marketing Budget Optimisation**

### Role

**Marketing Data Analyst / Marketing Analytics Consultant**

### Objective

Build an auditable analytics solution that connects customer marketing journeys to channel attribution and marketing investment analysis.

### What This Project Demonstrates

This project demonstrates the ability to move beyond simply creating charts.

It shows the ability to:

```text
Understand the business problem
        ↓
Audit the raw data
        ↓
Clean and validate the data
        ↓
Build analytical models
        ↓
Validate the calculations
        ↓
Build a semantic model
        ↓
Create DAX measures
        ↓
Design stakeholder dashboards
        ↓
Communicate business implications
```

This makes the project representative of a real-world **Marketing Analytics / Business Intelligence workflow**.

---

# 🚀 Potential Future Enhancements

Possible future extensions include:

- Shapley value attribution
- Markov-chain attribution
- Incrementality testing
- Marketing Mix Modeling
- Budget allocation simulation
- Scenario analysis
- Channel saturation modeling
- Customer lifetime value integration
- Automated Power BI refresh
- Marketing data warehouse integration
- Campaign-level experimentation

These are intentionally outside the current validated five-model implementation.

---

# 👩🏽‍💻 Author

## Patience Anono

**Marketing & Data Analytics Consultant**  
**PA Data Analytics**

I build data-driven analytics solutions that help organizations understand:

- Marketing performance
- Customer behavior
- Revenue drivers
- Channel contribution
- Business performance

### Portfolio

**Website:**  
https://padataanalytics.com

**GitHub:**  
https://github.com/PatienceAnono

---

# 📎 Repository

**GitHub Repository:**

https://github.com/PatienceAnono/multi-touch-attribution-budget-optimisation

---

# 📄 Disclaimer

This repository is a portfolio case study.

The attribution outputs represent **modeled allocations of observed conversion revenue** and should not be interpreted as causal estimates, guaranteed financial outcomes, or proof of incremental marketing impact.

The analysis is intended to demonstrate an analytical framework for understanding customer journeys, attribution methodology, channel performance, and marketing investment—not to provide financial guarantees or definitive causal conclusions.
