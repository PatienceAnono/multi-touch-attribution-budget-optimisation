# Multi-Touch Attribution & Marketing Budget Optimisation

> **An end-to-end marketing analytics project combining Python, attribution modeling, and Power BI to understand customer journeys, channel contribution, and marketing investment efficiency.**

**Built by PA Data Analytics**

---

## 📊 Project Overview

Marketing teams often struggle to answer a simple but important question:

> **Which marketing channels are actually contributing to conversions, and how should marketing investment be evaluated across the customer journey?**

A customer may interact with several channels before converting. Depending on how revenue is assigned, the same customer journey can tell very different stories.

This project builds a complete **multi-touch attribution and marketing performance analytics pipeline**, starting with raw customer-journey data and ending with an interactive, stakeholder-ready Power BI dashboard.

The project demonstrates the complete workflow:

**Raw Data → Data Cleaning → Data Validation → Attribution Modeling → Reconciliation → Power BI Modeling → DAX → Stakeholder Dashboard**

---

# 🎯 Business Objectives

The project was designed to answer five core business questions:

1. **How much conversion revenue is attributed to each marketing channel?**
2. **How does channel contribution change across attribution models?**
3. **Which channels and campaigns appear efficient relative to marketing spend?**
4. **What does the customer journey look like before conversion?**
5. **How should attribution insights be interpreted when making marketing investment decisions?**

---

# 📁 Dataset

The project uses customer-level marketing journey data containing multiple touchpoints leading to conversion or non-conversion.

### Validated Dataset

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

The cleaned journey dataset retains both converted and non-converted journeys.

The attribution output is restricted to converted journeys because revenue attribution requires a conversion outcome.

---

# 🧹 Data Cleaning & Quality Assurance

Before building the attribution models, the raw dataset was systematically audited.

## Duplicate Investigation

The raw data contained:

- **99 duplicate journey-position groups**
- **198 duplicate rows**
- Every duplicate group contained exactly two records
- Both records were substantively identical
- Differences were limited to tracking-level fields such as `touchpoint_id`
- Both records were flagged as duplicates
- No conflicting duplicate groups were found

### Canonical Deduplication Rule

Because every duplicate pair represented the same substantive touchpoint, the cleaning pipeline retained:

> **The lowest `touchpoint_id` as the deterministic canonical record.**

This removed **99 duplicate rows**, rather than incorrectly removing both records from each group.

### Validation Results

| Check | Result |
|---|---:|
| Journeys preserved | **3,500 / 3,500** |
| Complete touchpoint sequences | **100%** |
| Converted journeys protected | **1,454 / 1,454** |
| Revenue before cleaning | **$229,294.41** |
| Revenue after cleaning | **$229,294.41** |
| Revenue difference | **$0.00** |

The cleaning process therefore preserved the commercial integrity of the dataset.

---

# 🏷️ Channel Classification

Channel categories were re-derived using the project's established channel-to-category mapping.

The analysis classified:

- Paid Search → Paid
- Paid Social → Paid
- Display → Paid
- Influencer → Paid
- Email → Owned
- SEO/Organic → Earned
- Direct → Earned

This process corrected **18 inconsistent channel-category records**.

---

# 📈 Attribution Modeling

Five attribution models were implemented from scratch at the touchpoint level.

The models were applied to the **1,454 converted journeys**.

## 1. First-Touch Attribution

100% of conversion credit is assigned to the first touchpoint.

### Business interpretation

Useful for understanding:

- Customer acquisition
- Discovery
- Top-of-funnel channels

---

## 2. Last-Touch Attribution

100% of conversion credit is assigned to the final touchpoint before conversion.

### Business interpretation

Useful for understanding:

- Closing interactions
- Lower-funnel channels
- Conversion-stage activity

However, Last Touch can over-credit channels that appear immediately before purchase.

---

## 3. Linear Attribution

Conversion revenue is distributed equally across all touchpoints in a journey.

### Business interpretation

Provides a neutral multi-touch baseline where every touch receives equal credit.

---

## 4. Time-Decay Attribution

More recent interactions receive greater attribution weight.

The project uses a:

### **3-day half-life**

This was derived from the dataset's observed median conversion window of approximately six days.

The parameter is therefore explicitly documented as an analytical assumption rather than a fitted causal parameter.

---

## 5. Position-Based Attribution

The model uses a:

- **40%** first-touch allocation
- **20%** middle-touch allocation
- **40%** last-touch allocation

Special handling was applied for shorter journeys:

- 1 touchpoint → **100%**
- 2 touchpoints → **50% / 50%**

This avoids artificially assigning middle-touch credit where no middle touch exists.

---

# ✅ Attribution Reconciliation

All five attribution models successfully reconcile to the source conversion revenue.

| Attribution Model | Total Attributed Revenue |
|---|---:|
| First Touch | **$229,294.41** |
| Last Touch | **$229,294.41** |
| Linear | **$229,294.41** |
| Time Decay | **$229,294.41** |
| Position Based | **$229,294.41** |

### Validation

**1,454 / 1,454 converted journeys passed validation for every model.**

All models passed:

- Weight sum = 1
- Attribution revenue = journey revenue
- No negative weights
- No over-attribution
- No post-conversion touchpoints
- No duplicate attribution records

This produced:

> **$0.00 reconciliation difference across all five models.**

---

# 🔍 Key Attribution Findings

The most important finding is that **channel performance changes substantially depending on the attribution methodology used.**

| Channel | First Touch | Last Touch | Interpretation |
|---|---:|---:|---|
| Paid Search | 14.2% | **35.4%** | Strong closing role |
| Paid Social | **27.8%** | 14.7% | Stronger discovery role |
| Influencer | **19.2%** | 7.7% | Stronger top-of-funnel role |
| Email | 9.1% | **19.1%** | Stronger nurture/closing role |

### Business Interpretation

**Paid Search**

Paid Search increases significantly under Last Touch, suggesting that it frequently appears close to conversion.

**Paid Social**

Paid Social receives substantially more credit under First Touch than Last Touch, suggesting a stronger role earlier in the customer journey.

**Influencer**

Influencer shows a similar pattern, receiving considerably more credit under First Touch.

**Email**

Email gains attribution share toward Last Touch, suggesting an important nurture and closing role.

### Main insight

> **There is no single attribution view that tells the complete marketing story.**

The model comparison is therefore more valuable than simply selecting one "winning" attribution model.

---

# 🚦 Direct Traffic Analysis

Direct traffic was investigated before being included in the analysis.

Its journey-position distribution was:

| Position | Share |
|---|---:|
| First Touch | **27.6%** |
| Middle Touch | **37.3%** |
| Last Touch | **39.2%** |

Because Direct was not overwhelmingly concentrated in one position, it was retained as a genuine channel.

A separate Direct-excluded perspective was also produced for comparison.

Importantly, fully Direct journeys were excluded rather than artificially transferring their revenue to another channel.

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

Where a channel has zero recorded spend, ROAS is intentionally returned as blank rather than creating a misleading efficiency metric.

---

# 📊 Power BI Dashboard

The final analytical outputs were developed into a stakeholder-facing **five-page Power BI dashboard**.

---

## 1️⃣ Executive Overview

### Purpose

Provide leadership with a quick overview of marketing performance.

### Includes

- Total Conversion Revenue
- Total Marketing Spend
- ROAS
- Conversion Rate
- Converted Journeys
- Distinct Customers
- Monthly attributed revenue and marketing spend
- Channel attributed revenue ranking
- Executive interpretation

### Business Question

> **How is marketing performing overall, and where is attributed value concentrated?**

---

# 2️⃣ Attribution Analysis

### Purpose

This is the primary analytical page.

### Includes

- First Touch vs Last Touch comparison
- First Touch revenue
- Last Touch revenue
- Linear revenue
- Time Decay revenue
- Position Based revenue
- Channel attribution matrix
- Absolute attributed revenue comparison
- Interpretation callouts

### Business Question

> **How does the channel story change depending on the attribution methodology?**

---

# 3️⃣ Channel & Campaign Performance

### Purpose

Evaluate marketing efficiency across channels and campaigns.

### Includes

- Marketing Spend vs Attributed Revenue
- Channel ROAS
- Campaign ranking
- Efficiency matrix
- Cost per conversion
- Channel filter
- Campaign filter

### Business Question

> **Which channels and campaigns appear most efficient relative to marketing spend?**

---

# 4️⃣ Customer Journey

### Purpose

Understand how customers move through the marketing journey before converting.

### Includes

- Touchpoints per journey
- Days to conversion
- First-touch vs last-touch channel comparison
- Average touchpoints per journey
- Average days to conversion
- Revenue per converted customer
- Customer segment filtering
- Regional filtering

### Business Question

> **What does the customer journey look like before conversion?**

---

# 5️⃣ Marketing Investment

### Purpose

Connect marketing investment with attributed value.

### Includes

- Marketing Spend vs Attributed Revenue
- ROAS across attribution models
- Cost per conversion
- Channel efficiency
- Investment interpretation

### Business Question

> **Where is marketing investment producing attributed value, and how sensitive is the conclusion to the attribution model?**

---

# 🏗️ Power BI Architecture

The project deliberately separates analytical computation from reporting.

### Python

Used for:

- Data cleaning
- Duplicate investigation
- Journey validation
- Attribution calculations
- Revenue reconciliation
- Cost and ROI analysis

### Power BI

Used for:

- Semantic modeling
- Date dimension
- Channel dimension
- DAX measures
- Interactive filtering
- Visualization
- Executive reporting

The five attribution models are calculated in Python and then consumed by Power BI.

This creates an auditable analytical pipeline:

```text
RAW DATA
   ↓
DATA CLEANING
   ↓
DATA VALIDATION
   ↓
ATTRIBUTION MODELING
   ↓
RECONCILIATION
   ↓
POWER BI DATA MODEL
   ↓
DAX
   ↓
STAKEHOLDER DASHBOARD
