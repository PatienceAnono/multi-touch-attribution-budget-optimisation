# POWER BI VISUAL DESIGN SPECIFICATION
**Phase 5 — Stakeholder Dashboard Visual Design (DESIGN ONLY — no PBIR/visuals built)**

Marketing Attribution & Performance Analytics — PA Data Analytics portfolio project.

This spec is the design contract for Phase 6 (`powerbi-report-authoring`). It was produced by inspecting the validated Phase 4 semantic model directly (2 fact tables, `Date`, `DimChannel`, 43 measures) — every visual below references only fields/measures confirmed to exist. No new DAX is introduced; two report-layer-only techniques (native numeric binning, "show value as % of grand total") are used instead of new measures and are flagged explicitly where used.

---

## 1. Report Purpose

Show how marketing touchpoints across the customer journey connect to the **$229,294.41** in validated conversion revenue — and how that story changes depending on which of five attribution models is used to divide credit. The report answers five distinct business questions, one per page, and is built to be presented live to stakeholders who did not build it.

**Design identity**
- **Tone: Brand-Forward (PA Data Analytics)**, structured on the *Corporate Cool* pattern — a cool, light, restrained B2B surface is the closest documented tone to PA's navy/blue/light palette, adapted with PA's exact hues rather than Corporate Cool's stock slate/cyan.
  - Surface: Light Background `#F8F9FC`; visual containers White `#FFFFFF` with a 1px hairline border `#E2E8F0`, 8px corner radius.
  - Type: Segoe UI throughout (PBI-native, renders consistently across Desktop/Service). Titles in PA Navy `#272D78`; body/axis/labels in Dark Text `#555770`.
  - Density ratio: 1.333 — restrained hierarchy, comfortable for a mixed exec/analyst audience.
  - Gridlines: solid, low-opacity `#E2E8F0`. No visual borders beyond the hairline card rule; no drop shadows.
- **Signature — "PA Highlight Discipline":** every chart follows highlight-and-grey using PA's own hues, and every measure keeps one fixed color across the whole report. **PA Blue `#3386C7`** is the single primary accent — the channel/series/bar currently under discussion (or the default "current value" series) is always Blue; everything else is neutral grey (`#BDBDBD`/`#9CA3AF`). **PA Navy `#272D78`** carries titles, the "prior/reference" series in paired comparisons, and KPI-card top accent bars. **Teal `#2EC4B6`** is reserved exclusively for one positive/secondary emphasis per page (e.g., the single best-ROAS channel) — it is never a default series color. This recurs on all 5 pages and makes the report instantly identifiable as PA Data Analytics without touching the logo.

---

## 2. Target Stakeholders

| Audience | What they need from this report |
|---|---|
| Marketing managers | Which channels/campaigns to fund, and how attribution-model choice changes that answer |
| E-commerce managers | How customers move through the funnel before buying; where journeys stall or convert fast |
| Founders / senior stakeholders | A 10-second read on whether marketing spend is producing revenue, without needing to understand attribution theory |
| (Portfolio reviewers) | Evidence of rigorous, non-causal, model-aware analytical thinking — language and visuals must never overstate certainty |

---

## 3. Page Structure

The user's proposed 5-page structure is sound and is kept as-is; each page was independently routed to an archetype using its own data shape (not copied from page 1), per design-system best practice for multi-page reports.

| # | Page | Archetype | Layout Variant | Why this variant |
|---|---|---|---|---|
| 1 | Executive Overview | Executive Summary | **B — KPI-Strip** | 6 KPIs of comparable importance (Revenue, Spend, ROAS, Conversion Rate, Journeys, Customers) with no single dominant hero metric |
| 2 | Attribution Analysis (hero page) | Comparative Benchmark | **C — Slope-Graph-First** | The core question is a **rank shift** (channels move up/down between First-Touch and Last-Touch) — position-on-shared-scale is exactly what a slope graph shows and a bar chart cannot |
| 3 | Channel & Campaign Performance | Analytical Canvas | **B — Inline-Slicers** | Only 3 slicers needed (Date, Channel, Campaign); a scatter + two rankings need full page width, not a filter rail |
| 4 | Customer Journey | Analytical Canvas | **B — Inline-Slicers** | Distributional/exploratory like Page 3, but deliberately uses a *different* visual mix (distributions + paired bar, no scatter) so the two Analytical pages don't feel identical; 3 slicers (Date, Segment, Region) |
| 5 | Marketing Investment | Comparative Benchmark | **A — Side-by-Side** | The question is "spend share vs. value share by channel" — a headline variance comparison with small-multiples-style ROAS-by-model support, distinct from Page 2's rank-shift framing and Page 3's efficiency-ranking framing |

Cross-page rotation check: Pages 2 & 5 share the Comparative archetype but use different variants (C vs A); Pages 3 & 4 share Analytical Canvas but use different visual mixes to avoid feeling templated. No two pages are visually interchangeable.

**Global slicer:** one synced **Date range** slicer (on `Date[Date]`, Between mode) appears top-right on all 5 pages — the only element that repeats everywhere, because every page's story can shift by month and the data spans a full year (Jan–Dec 2024) with real month-to-month variation. No other slicer repeats on every page (see §6).

---

## 4. Page-by-Page Visual Specifications

### PAGE 1 — Executive Overview
**Business question:** "How is marketing performing overall?"
**What the stakeholder should learn:** At a glance, whether marketing is converting spend into revenue, and which channel currently earns the most attributed credit.
**What decision this page supports:** Whether overall marketing investment looks healthy enough to not need deeper investigation this cycle — if not, which page to open next.

Layout: Variant B (KPI-Strip). Title bar (h=48) + subtitle → 6-card KPI strip (h=120, full width) → analysis row (h=300: trend chart | ranked bar) → footer.

**KPI cards (left → right):**

| # | Measure | Format | Comparison period? | Variance shown? | Target? |
|---|---|---|---|---|---|
| 1 | `Total Conversion Revenue` | `$#,##0` | No — single year of data, no prior-year baseline exists | No | No — no target exists in the data; do not invent one |
| 2 | `Total Marketing Spend` | `$#,##0` | No | No | No |
| 3 | `ROAS Linear` (labeled "Overall ROAS — Linear Model") | `0.00×` | No | No | No |
| 4 | `Conversion Rate` | `0.0%` | No | No | No |
| 5 | `Converted Journeys` | `#,##0` | No | No | No |
| 6 | `Distinct Customers` | `#,##0` | No | No | No |

No period-over-period deltas anywhere in this report: the dataset covers one calendar year with no prior-year table, so a "vs last period" arrow would have to be invented. This is stated explicitly rather than fabricated — a deliberate, evidence-based deviation from the generic Executive-Summary template (which normally mandates deltas).

> Why "ROAS Linear" and not a generic "ROAS": no single blended-ROAS measure exists in the model (by design — ROAS is defined per attribution model, since "attributed revenue" is model-dependent). Linear is used as the neutral, easy-to-explain executive default and is labeled to say so; Pages 2–5 show all 5 models.

**Visual 1 — Revenue & Spend Trend**
- Business question: Is monthly revenue tracking with, or diverging from, spend?
- Type: `lineChart`, 2 series (same unit, $ — safe to combine per chart-selection rules; never dual-axis)
- Fields: Axis `Date[YearMonth]`; Values `Total Conversion Revenue`, `Total Marketing Spend`
- Legend: series names, top
- Filters: none page-specific (respects global Date slicer)
- Sorting: chronological (YearMonth ascending)
- Conditional formatting: none (2-line comparison doesn't need CF)
- Colors: `Total Conversion Revenue` = PA Blue `#3386C7` (primary accent — this is the measure every KPI card and page opens with); `Total Marketing Spend` = PA Navy `#272D78` (reference series)
- Title: "Revenue Held Steady While Spend Tracked Closely" *(exact wording finalized against the live chart shape in Phase 6 — see note below)*
- Subtitle: "Monthly attributed revenue vs. marketing spend, 2024"
- Tooltip: Month, Total Conversion Revenue, Total Marketing Spend
- Why it matters: proves the KPI-card headline with an actual trend, per Executive-Summary principle #5 (one hero chart proving the headline)
- Expected takeaway: revenue and spend move together for most of the year; a reader should be able to spot any month where they diverge

> **Titling note (applies to every "insight" title in this spec):** several titles below state a data-driven headline (e.g., "Paid Search Closes More Than It Opens"). These are *placeholder theses built from the numbers already validated in Phase 3* — Phase 6 must re-verify the exact wording against the live rendered chart (sign, magnitude, ranking) before publishing, per the "descriptive title, not a label" principle. None should be typeset without that final check.

**Visual 2 — Channel Performance (Linear Model)**
- Business question: Which channels currently hold the most attributed revenue?
- Type: `barChart`, horizontal, sorted descending
- Fields: Axis `DimChannel[channel]`; Value `Linear Revenue`
- Filters: none page-specific
- Sorting: descending by `Linear Revenue`
- Conditional formatting: light→dark gradient of PA Blue keyed to `Linear Revenue` magnitude (breakdown-of-a-measure gradient rule)
- Title: "Which Channels Hold the Most Attributed Revenue"
- Subtitle: "Linear model — equal credit across every touchpoint in the journey"
- Tooltip: Channel, Linear Revenue, ROAS Linear, First Touch Revenue, Last Touch Revenue
- Why it matters: gives the one ranked view every stakeholder expects on an exec page, using the most "average"/defensible model
- Expected takeaway: Paid Social and Paid Search are the two largest channels under a neutral model; full nuance is deferred to Page 2

**Executive insight callout** (textbox, not a chart): one sentence, e.g. *"Attribution model choice changes which channel looks strongest — see Attribution Analysis for the full picture."* This is the page's bridge to the hero page, and its `callout_value_basis` is the cross-model variance already computed by `Model Revenue Spread` (not a repeated absolute number).

---

### PAGE 2 — Attribution Analysis (HERO PAGE)
**Business question:** "How does channel contribution change depending on the attribution model?"
**What the stakeholder should learn:** The same $229,294.41 gets credited very differently depending on modeling choice — Paid Search and Email look like closers, Paid Social and Influencer look like openers, and no single "right" answer exists.
**What decision this page supports:** Which attribution model (or blend of perspectives) to adopt as the organization's reporting standard, with eyes open about what each model emphasizes.

Layout: Comparative Benchmark, Variant C (Slope-Graph-First). Title bar → hero slope graph (h≈420) → supporting small multiples (h≈220) → matrix + ranked bar row → insight callout → footer.

**Visual 1 (HERO) — First-Touch vs. Last-Touch: How Channel Rank Shifts**
- Business question: Which channels gain or lose standing depending on whether you credit the first or last touchpoint?
- Type: `lineChart` simulating a slope graph — categorical X axis with exactly 2 points ("First Touch", "Last Touch"), one line per channel
- Fields: Axis: a 2-value category (First Touch / Last Touch); Values: `First Touch Revenue`, `Last Touch Revenue`, one series per `DimChannel[channel]`
- Legend: channel names, right side, ordered by Last-Touch value
- Filters: respects Channel slicer (see §6) and global Date
- Sorting: n/a (position on the two axes is the point)
- Conditional formatting: **highlight-and-grey** — Paid Search, Paid Social, Influencer, and Email (the four channels with a validated, meaningful shift) rendered in PA Blue with distinct line styles/end-labels; Direct, SEO/Organic, and Display rendered in flat grey `#BDBDBD` as context
- Title: "Paid Search and Email Gain Ground as Closers; Paid Social and Influencer Lose Ground"
- Subtitle: "Attribution Share by touchpoint position — First Touch vs. Last Touch, not causal impact"
- Tooltip per line: Channel, First Touch Revenue, First Touch Share %, Last Touch Revenue, Last Touch Share %
- Why it matters: this is the single visual that most directly answers the page's business question — rank/position change, not just magnitude
- Expected takeaway: Paid Search (14.2%→35.4% share) and Email (9.1%→19.1%) visibly rise; Paid Social (27.8%→14.7%) and Influencer (19.2%→7.7%) visibly fall — read as "closing power" and "discovery power," never as "caused"

**Visual 2 — Attribution Model Comparison by Channel**
- Business question: Does every model tell a different story, or do some models roughly agree?
- Type: `smallMultiplesChart` — 5 panels (one per model), each a sorted horizontal bar of channel revenue, **shared Y axis mandatory** (0 to ~$90K, so bar length is honestly comparable across panels)
- Fields: Panel: a static list of the 5 model names (via 5 separate panels bound to `First Touch Revenue`, `Last Touch Revenue`, `Linear Revenue`, `Time Decay Revenue`, `Position Based Revenue` respectively — 5 existing measures, no field parameter/new DAX needed); Axis (per panel): `DimChannel[channel]`
- Filters: respects Channel/Date slicers
- Sorting: each panel sorted independently by its own value, descending
- Conditional formatting: PA Blue gradient per panel (same rule as Page 1 Visual 2)
- Title: "Five Models, Five Different Rankings"
- Subtitle: "Attributed revenue by channel, shown separately for each model"
- Tooltip: Channel, model name, Attributed Revenue, Attribution Share %
- Why it matters: this is the direct visual proof that "different models tell different stories," which the brief explicitly requires demonstrating
- Expected takeaway: Paid Social tops First-Touch; Paid Search tops Last-Touch; Linear/Time-Decay/Position-Based sit visibly between the two extremes

**Visual 3 — Channel Attribution Matrix**
- Business question: What is the exact attributed-revenue number for each channel under each model?
- Type: `matrix` (`tableEx`/pivot)
- Rows: `DimChannel[channel]`; Columns: `First Touch Revenue`, `Last Touch Revenue`, `Linear Revenue`, `Time Decay Revenue`, `Position Based Revenue`, `Model Revenue Spread`
- Filters: respects Channel/Date slicers
- Sorting: rows sorted by `Linear Revenue` descending (a neutral, defensible default order)
- Conditional formatting: color-scale (sequential Blues) on the `Model Revenue Spread` column only — this is the one column that answers "which channel's story changes most depending on the model," which is exactly the page's business question; data bars on the 5 revenue columns for at-a-glance magnitude
- Title: "The Exact Numbers Behind Every Model"
- Subtitle: "Attributed revenue by channel and model — see Model Revenue Spread for how much the story changes"
- Tooltip: n/a (matrix cells are precision-first; no additional tooltip needed)
- Why it matters: analysts and precision-seeking stakeholders need exact figures, not just a chart shape; the Spread column converts the whole matrix into a ranked "sensitivity" view for free
- Expected takeaway: Paid Search and Paid Social show the largest spread (most model-sensitive); Direct and SEO/Organic show the smallest (attribution model choice barely matters for them)

**Visual 4 — Current Ranking, Absolute Scale (supporting the slope graph)**
- Business question: What's the actual dollar ranking right now, independent of the rank-shift story?
- Type: `barChart`, horizontal, sorted descending
- Fields: Axis `DimChannel[channel]`; Value `Linear Revenue`
- Filters: respects Channel/Date slicers
- Sorting: descending
- Conditional formatting: none (this is the deliberately "plain" reference visual — the slope graph carries the color story)
- Title: "Absolute Scale: Where Each Channel Stands Today"
- Subtitle: "Linear model, for reference — the slope graph above hides absolute dollar values by design"
- Tooltip: Channel, Linear Revenue
- Why it matters: per the Comparative-Benchmark archetype, a slope graph intentionally hides magnitude to show rank-shift clearly; this bar restores the missing scale
- Expected takeaway: confirms Paid Social and Paid Search are the two largest channels in absolute terms, contextualizing visual 1's rank-shift story

**Insight callout:** "Paid Search and Email look strongest when you credit the *closing* touch; Paid Social and Influencer look strongest when you credit the *opening* touch. Neither is 'more correct' — they answer different questions about the same $229,294.41." — directly operationalizes the brief's non-causality requirement.

---

### PAGE 3 — Channel & Campaign Performance
**Business question:** "Which channels and campaigns are delivering efficient performance?"
**What the stakeholder should learn:** Efficiency (revenue per dollar spent) does not always track with raw revenue size — some smaller channels/campaigns punch above their weight.
**What decision this page supports:** Where to shift incremental budget among currently-active channels/campaigns.

Layout: Analytical Canvas, Variant B (Inline-Slicers). Title row + 3 inline slicers → hero scatter (h=320, full width) → 2 supporting rankings (h=200) → efficiency matrix (h=240) → footer.

**Visual 1 (HERO) — Spend vs. Attributed Revenue by Channel**
- Business question: Is spend proportional to the revenue a channel receives credit for?
- Type: `scatterChart` — evaluated and recommended: with only 7 channels a bar chart could rank each metric separately, but only a scatter shows *both* metrics' relationship in one glance and makes efficiency (a steep spend→revenue slope) visually obvious without a manufactured reference line
- Fields: X = `Total Marketing Spend`; Y = `Linear Revenue`; Size = `ROAS Linear`; Category (color + label) = `DimChannel[channel]`
- Legend: off — with only 7 points, direct data labels on each point are clearer than a legend (chart-selection rule: label directly when ≤4... here 7 points still reads fine labeled since it's not a dense scatter)
- Filters: respects page slicers
- Sorting: n/a
- Conditional formatting: bubble color = PA Blue for all points except the single highest-ROAS channel, which gets Teal (the page's one reserved positive-emphasis use)
- Title: "Efficiency Isn't Always Where the Spend Is"
- Subtitle: "Bubble size = ROAS (Linear model); each point is one channel"
- Tooltip: Channel, Total Marketing Spend, Linear Revenue, ROAS Linear, Cost per Conversion Linear
- Why it matters: directly answers the brief's explicit ask to evaluate a scatter for this exact comparison — recommended because it's the only chart type that encodes the *relationship*, not just either metric alone
- Expected takeaway: Email and Paid Search sit high-revenue/low-spend (efficient); Influencer sits higher-spend for its revenue (least efficient on a per-dollar basis, still valuable in absolute terms per Page 2)

**Visual 2 — Channel Ranking by Efficiency**
- Type: `barChart`, horizontal, sorted descending
- Fields: Axis `DimChannel[channel]`; Value `ROAS Linear`
- Conditional formatting: sequential Blues gradient by magnitude (not semantic green/red — no defined ROAS target exists in the data, so no "good/bad" claim is made, only a ranked pattern)
- Title: "Channels Ranked by Return per Dollar Spent"
- Subtitle: "ROAS, Linear model"
- Tooltip: Channel, ROAS Linear, Linear Revenue, Total Marketing Spend
- Why it matters: gives the direct ranked answer the scatter only implies visually
- Expected takeaway: confirms the scatter's efficient/inefficient read in an unambiguous sorted list

**Visual 3 — Top 10 Campaigns by Attributed Revenue**
- Type: `barChart`, horizontal, sorted descending, Top-N filter = 10 (campaign_name has 32 distinct values — well above the 15–20 cardinality ceiling for a readable bar chart, so a Top-10 filter is applied rather than plotting all 32)
- Fields: Axis `Attribution[campaign_name]`; Value `Linear Revenue`
- Filters: page-level Campaign slicer available for stakeholders who want a specific campaign instead of the Top-10 default
- Sorting: descending
- Conditional formatting: PA Blue gradient
- Title: "Top 10 Campaigns by Attributed Revenue"
- Subtitle: "Linear model; 32 campaigns tracked in total, plus untracked (No Campaign) traffic"
- Tooltip: Campaign, Linear Revenue, First Touch Revenue, Last Touch Revenue
- Why it matters: campaign-level detail the exec page deliberately omits; Top-N keeps it scannable
- Expected takeaway: identifies which named campaigns are actually driving the channel totals from Visual 2

**Visual 4 — Channel Efficiency Detail**
- Type: `matrix`
- Rows: `DimChannel[channel]`; Columns: `Total Marketing Spend`, `Linear Revenue`, `ROAS Linear`, `Cost per Conversion Linear`
- Conditional formatting: data bars on Spend and Revenue; sequential color scale on ROAS
- Title: "The Exact Numbers Behind the Efficiency Story"
- Tooltip: n/a (matrix)
- Why it matters: gives the precise figures behind Visuals 1–2 for stakeholders who want to check the math
- Expected takeaway: same ranking as Visual 2, now with exact dollar figures for budget conversations

---

### PAGE 4 — Customer Journey
**Business question:** "How do customers move through the marketing journey before converting?"
**What the stakeholder should learn:** Most converting journeys are short (1–3 touchpoints, days not weeks), and the channel that opens a journey is frequently different from the one that closes it.
**What decision this page supports:** Whether the funnel is behaving as expected (short, fast journeys) or whether specific segments/regions show unusually long paths worth investigating.

Layout: Analytical Canvas, Variant B (Inline-Slicers). Title row + 3 inline slicers (Date, Customer Segment, Region) → 2 distribution charts (h=260) → paired first/last-touch bar (h=260) → KPI strip + callout → footer. No Sankey/funnel/custom visual is used — every visual below is a native Power BI type, per the brief's explicit instruction.

**Visual 1 — How Many Touchpoints Before Converting**
- Type: `columnChart` (touchpoint count only ranges 1–7, so no binning is needed — each value is already a clean category)
- Fields: Axis `attribution_customer_journeys[total_touchpoints]`; Value `Converted Journeys` (uses the existing measure, which already filters to `journey_converted = 1`)
- Filters: respects page slicers
- Sorting: ascending by touchpoint count (meaningful order, not alphabetical)
- Conditional formatting: PA Blue gradient
- Title: "Most Converting Journeys Take Three Touchpoints or Fewer"
- Subtitle: "Number of converted journeys by total touchpoint count"
- Tooltip: Touchpoint Count, Converted Journeys, % of all converted journeys
- Why it matters: directly answers "how many touchpoints" from the brief
- Expected takeaway: the distribution is front-loaded — few journeys need more than 3–4 touches to convert

**Visual 2 — How Long Journeys Take to Convert**
- Type: `columnChart` with **native Power BI numeric binning** applied to `days_to_conversion` (bin size ≈ 5 days; report-authoring field-grouping feature, not a new DAX measure — days_to_conversion ranges 1–30, too wide to plot as 30 raw categories per the cardinality guidance)
- Fields: Axis `attribution_customer_journeys[days_to_conversion]` (binned); Value `Converted Journeys`
- Sorting: ascending by day bucket
- Conditional formatting: PA Blue gradient
- Title: "Most Conversions Happen Within Two Weeks"
- Subtitle: "Converted journeys by days from first touch to conversion (median 6 days)"
- Tooltip: Day range, Converted Journeys
- Why it matters: directly answers "time to conversion" from the brief, reusing the already-validated 6-day median from Phase 3
- Expected takeaway: the distribution right-skews — a long tail of slower conversions exists but is not typical

**Visual 3 — First-Touch Channel vs. Last-Touch Channel**
- Type: `clusteredBarChart`, horizontal, 2 series
- Fields: Axis `attribution_customer_journeys[channel]`; Values: two implicit `COUNT` aggregations of `journey_id` (distinct), each with a **visual-level filter** — Series A filters `is_first_touch = 1`, Series B filters `is_last_touch = 1`. *(This uses existing columns with a report-level filter, not a new DAX measure — flagged per the QA requirement below.)*
- Legend: "Opens the Journey" (Navy) / "Closes the Journey" (Blue)
- Sorting: descending by the "Closes the Journey" series
- Conditional formatting: fixed 2-series colors as above (not magnitude-based, since the comparison IS the two series)
- Title: "Which Channels Open Journeys vs. Which Ones Close Them"
- Subtitle: "Count of journeys where the channel was the first vs. last touchpoint (all journeys, not only converted)"
- Tooltip: Channel, Opens count, Closes count
- Why it matters: this is the volume-based (journey-count) companion to Page 2's dollar-based rank-shift story — confirms the same discovery/closing pattern holds at the raw behavioral level, not only in attributed dollars
- Expected takeaway: mirrors Page 2's finding — Paid Social/Influencer skew toward opening, Paid Search/Email skew toward closing

**KPI strip (3 compact cards, not the page hero):**

| Measure | Format | Notes |
|---|---|---|
| `Avg Touchpoints per Journey` | `0.00` | Context for Visual 1 |
| `Avg Days to Conversion` | `0.0 days` | Context for Visual 2 |
| `Revenue per Converted Customer (Linear)` | `$#,##0` | Ties journey behavior back to value, closing the page's loop |

**Insight callout:** "Journeys are short and fast for most customers — but the channel that starts a journey is often not the one that gets credited at the end, echoing the Attribution Analysis findings."

---

### PAGE 5 — Marketing Investment
**Business question:** "Where is marketing investment producing attributed value?"
**What the stakeholder should learn:** Some channels take a bigger share of the budget than they return in attributed value, and vice versa — and this "attributed value" is a *shared credit* against one real revenue pool, not five separate pots of money.
**What decision this page supports:** Budget reallocation between over- and under-indexed channels.

Layout: Comparative Benchmark, Variant A (Side-by-Side). Title row → headline paired-share chart (h=320, full width) → ROAS-by-model matrix + cost-efficiency ranking (h=260, 2-up) → insight callout → footer.

**Explicit distinction (stated in the page subtitle, not just this spec):** `Total Marketing Spend` is real money spent. `Total Conversion Revenue` ($229,294.41) is the one real pool of actual conversion revenue. Every "Attributed Revenue" figure on this page (First/Last/Linear/Time-Decay/Position-Based) is that *same* $229,294.41 redistributed across channels under a modeling rule — channels do not each separately generate their attributed figure, they share credit for one outcome. This is stated explicitly to prevent the single most common misreading of an attribution dashboard.

**Visual 1 (HERO) — Spend Share vs. Revenue Share by Channel**
- Business question: Does each channel's slice of the budget match its slice of attributed value?
- Type: `clusteredBarChart`, 2 series, sorted by revenue share descending
- Fields: Axis `DimChannel[channel]`; Values: `Total Marketing Spend` and `Linear Revenue`, both displayed via Power BI's native **"Show value as → Percent of Grand Total"** (a report-authoring display option, not a new DAX measure — flagged per the QA requirement below)
- Legend: "Share of Spend" (Navy) / "Share of Attributed Value" (Blue)
- Filters: respects Channel/Date slicers
- Sorting: descending by Share of Attributed Value
- Conditional formatting: fixed 2-series colors (comparison is the point, not magnitude within a series)
- Title: "Which Channels Take More Budget Than Credit — and Which Take Less"
- Subtitle: "Share of total spend vs. share of total attributed revenue (Linear model)"
- Tooltip: Channel, Spend Share %, Revenue Share %, gap (Revenue Share − Spend Share)
- Why it matters: this is the one visual that directly answers "where is investment producing value," in relative terms that are comparable across channels of very different absolute size
- Expected takeaway: channels whose blue bar clearly exceeds their navy bar are out-punching their budget; the reverse signals potential overspend relative to attributed credit

**Visual 2 — ROAS Across All 5 Models**
- Type: `matrix`
- Rows: `DimChannel[channel]`; Columns: `ROAS First Touch`, `ROAS Last Touch`, `ROAS Linear`, `ROAS Time Decay`, `ROAS Position Based`
- Conditional formatting: sequential color-scale (Blues) per column
- Title: "Return on Spend Also Depends on the Attribution Model"
- Subtitle: "ROAS by channel; Direct and SEO/Organic show no ratio — they carry no tracked media cost"
- Tooltip: n/a (matrix)
- Why it matters: extends Page 2's "different models, different stories" finding into an investment-decision context; explicitly documents why 2 of 7 rows are blank (no fabricated ROAS for $0-spend channels, matching the Python methodology)
- Expected takeaway: Paid Search's ROAS is dramatically higher under Last-Touch than First-Touch — a budget conversation should specify *which* model it's using

**Visual 3 — Cost per Conversion by Channel**
- Type: `barChart`, horizontal, sorted ascending (lower = more efficient)
- Fields: Axis `DimChannel[channel]`; Value `Cost per Conversion Linear`
- Conditional formatting: PA Blue bars; the single lowest-cost channel highlighted in Teal (the page's one reserved positive-emphasis use)
- Title: "Cost to Earn a Credit-Weighted Conversion, by Channel"
- Subtitle: "Linear model; Direct and SEO/Organic show $0 — no media cost, not zero effort"
- Tooltip: Channel, Cost per Conversion Linear, Total Marketing Spend
- Why it matters: gives a single, comparable efficiency number per channel for budget prioritization
- Expected takeaway: identifies the cheapest and most expensive channels per conversion credit

**Insight callout:** "Attributed Revenue is not five separate revenue pools — it's the same $229,294.41 shared five different ways. Use the Spend vs. Attributed Value chart above to see which channels are over- or under-funded relative to the credit they currently receive."

---

## 5. KPI Specification (all cards, all pages)

| Page | KPI | Measure | Format | Comparison period | Variance shown | Target |
|---|---|---|---|---|---|---|
| 1 | Total Conversion Revenue | `Total Conversion Revenue` | `$#,##0` | None available | No | None — not in data |
| 1 | Total Marketing Spend | `Total Marketing Spend` | `$#,##0` | None available | No | None |
| 1 | Overall ROAS — Linear Model | `ROAS Linear` | `0.00×` | None available | No | None |
| 1 | Conversion Rate | `Conversion Rate` | `0.0%` | None available | No | None |
| 1 | Converted Journeys | `Converted Journeys` | `#,##0` | None available | No | None |
| 1 | Distinct Customers | `Distinct Customers` | `#,##0` | None available | No | None |
| 4 | Avg Touchpoints per Journey | `Avg Touchpoints per Journey` | `0.00` | None available | No | None |
| 4 | Avg Days to Conversion | `Avg Days to Conversion` | `0.0 "days"` | None available | No | None |
| 4 | Revenue per Converted Customer | `Revenue per Converted Customer (Linear)` | `$#,##0` | None available | No | None |

**No KPI anywhere in this report shows a period-over-period delta or a target/threshold.** The dataset is a single 2024 calendar year with no prior-year comparison table and no stated business targets — inventing either would violate the brief's explicit instruction not to invent targets that don't exist in the data. If a future data refresh adds a prior period or a stated goal, deltas/targets should be added at that point, not before.

---

## 6. Slicers

| Slicer | Field | Scope | Pages | Control type | Rationale |
|---|---|---|---|---|---|
| Date range | `Date[Date]` | **Global**, synced (`syncGroup`) | All 5 | Between (date range) | Only cut that's useful everywhere — 12 months of real variation exist; a Between control (not a Year dropdown) because sub-year granularity is where the variation lives |
| Channel | `DimChannel[channel]` | Page-level | 2, 3, 5 | Tile list (7 values, low cardinality) | Central dimension on the pages built around channel comparison; irrelevant filtering noise on 1 (exec, ≤1 slicer rule) and 4 (journey behavior, not channel-first) |
| Campaign | `Attribution[campaign_name]` | Page-level | 3 only | Dropdown with search (32 values, medium cardinality) | Only page with campaign-grain visuals |
| Customer Segment | `attribution_customer_journeys[customer_segment]` | Page-level | 4 only | Tile list (5 values) | Only page examining journey behavior by segment |
| Region | `attribution_customer_journeys[region]` | Page-level | 4 only | Tile list (5 values) | Companion cut to Segment on the one page built for journey-behavior exploration |

No page carries more than 3 slicers total (Page 3 and Page 4 each carry Date + 2 page-level = 3), staying well clear of the "wall of slicers" anti-pattern (4+ triggers a filter rail, which none of these pages need).

---

## 7. Interactions

- **Cross-filtering:** default Power BI cross-filter/highlight behavior stays on within each page (e.g., clicking a channel in Page 3's ranking bar highlights it in the scatter and matrix). No cross-filter links are disabled, since every visual on a page shares the same grain-compatible dimensions.
- **No bookmarks, drill-through, or navigation buttons are specified in this phase** — the brief for Phase 5 is design-only and the report is small enough (5 pages) that native Power BI page tabs are sufficient navigation; this can be revisited in Phase 6 if stakeholder testing shows a need.
- **No field parameters** are used (e.g., a "pick your attribution model" dropdown), because implementing one requires a small calculated table in the semantic model — out of scope for a phase that explicitly forbids new DAX/model changes unless "absolutely required and explicitly identified." Instead, all 5 models are shown side-by-side (small multiples, matrices) using only existing measures. **Recommendation for a future phase:** a Model Selector field parameter would meaningfully improve Page 2 and Page 5 by letting a stakeholder isolate one model at a time — flagged here as a specific, identified, optional DAX addition for Phase 6 sign-off, not built now.

---

## 8. Conditional Formatting Summary

| Where | Technique | Basis | Why |
|---|---|---|---|
| Bar charts breaking down one measure (Pages 1, 2, 3) | Light→dark gradient of PA Blue | The measure's own magnitude | Ties the breakdown visually to its parent measure (color.md gradient rule) |
| `Model Revenue Spread` column, Page 2 matrix | Sequential color scale (Blues) | Spread magnitude | Directly highlights which channels are most "model-sensitive" — the page's core question, in one column |
| ROAS columns/bars (Pages 3, 5) | Sequential color scale or gradient | ROAS magnitude | Pattern only, not "good/bad" — no ROAS target exists in the data, so no semantic green/red is used |
| Slope graph, Page 2 | Highlight-and-grey: PA Blue for Paid Search/Paid Social/Influencer/Email, grey for the rest | Whether the channel has a validated, meaningful rank-shift finding | Draws attention to the exact 4 channels the business findings are about, without discarding the other 3 as data |
| Single best-ROAS/lowest-cost point (Pages 3, 5) | One-off Teal highlight | Rank (#1 only) | The report's only reserved use of Teal — a single positive callout per page, never a default series color |
| First/last-touch paired bar, Page 4 | Fixed 2-series colors (Navy/Blue), not magnitude-based | Series identity | The comparison is between two series, not a magnitude ranking within one |

**Never used:** red/green semantic pairs (no validated target or benchmark exists anywhere in this dataset to justify "good" vs "bad"), gauges, pie/donut charts (all part-to-whole questions in this report have >5 slices — 7 channels, 5 models, or 32 campaigns — so sorted bars are used per the chart-selection matrix), 3D effects, dual-axis charts.

---

## 9. Tooltips

| Context | Tooltip fields |
|---|---|
| Any channel-level visual | Channel · Attributed Revenue (relevant model) · Spend (where applicable) · ROAS (where applicable) · First Touch Share % · Last Touch Share % |
| Any campaign-level visual | Campaign · Attributed Revenue (relevant model) · First Touch Revenue · Last Touch Revenue |
| Slope graph (Page 2) | Channel · First Touch Revenue · First Touch Share % · Last Touch Revenue · Last Touch Share % |
| Journey/time distributions (Page 4) | Bucket (touchpoint count or day range) · Converted Journeys · % of all converted journeys |

Kept short deliberately (4–5 fields max) per the brief's "do not overload tooltips" instruction — every tooltip answers "what am I looking at" and "how does it compare," nothing more.

---

## 10. Branding

| Token | Hex | Use |
|---|---|---|
| PA Navy | `#272D78` | Page/visual titles, KPI-card top accent bar, "reference" series in paired comparisons (e.g., Spend, Prior series) |
| PA Blue | `#3386C7` | Single primary accent — the "current value"/highlighted series in every chart; KPI values |
| White | `#FFFFFF` | Visual container background |
| Light Background | `#F8F9FC` | Page canvas |
| Dark Text | `#555770` | Body text, axis labels, tooltips, footnotes |
| Teal (optional) | `#2EC4B6` | Reserved — exactly one positive/secondary emphasis per page where used (Pages 3, 5); never a default series color |
| Neutral grey (report-defined, not in the brand sheet) | `#BDBDBD` / `#9CA3AF` | Context/de-emphasized series in every highlight-and-grey chart |

The PA Data Analytics logo is not altered and is placed once, top-left of Page 1 only (standard report cover placement) — no unrelated colors are introduced anywhere in the palette above.

---

## 11. Accessibility

- **Contrast — verified against WCAG AA, with one real constraint found:** Navy `#272D78` on White/Light-Background passes AAA for any text size. Dark Text `#555770` on `#F8F9FC` is close to the reference `#767676`-on-white pairing that just clears AA body text (4.5:1) — usable for body copy, but must be re-checked with a contrast tool at build time rather than assumed. **PA Blue `#3386C7` on white is estimated near 4.2:1** (based on the closest documented reference swatch, `#3182BD`) — this **passes for large text and non-text elements (KPI numerals ≥18pt, bar/line marks) but likely fails small body text at 4.5:1**. Consequence for Phase 6: PA Blue is used for KPI values, bars, and lines (all large/non-text) throughout this spec — it is never specified for axis labels, tooltip body text, or footnotes, which stay in Dark Text or Navy. Teal is treated the same way (large/non-text only) as a precaution, since it is likely similar or lower contrast than Blue.
- **Color is never the sole signal:** every highlight-and-grey chart pairs color with a direct data label or legend entry (channel name), not color alone — satisfies WCAG 1.4.1.
- **Alt text:** every visual gets insight-driven alt text at build time (e.g., "Paid Search's revenue share rises from 14.2% at first touch to 35.4% at last touch" rather than "line chart"), per the accessibility reference's alt-text templates. Not authored in this design-only phase, but required before Phase 6 sign-off.
- **Tab order:** Selection-pane tab order should follow each page's stated visual order (Hero → supporting → matrix → callout), matching reading flow.
- **Show as table:** left enabled (Power BI default) on every visual as the screen-reader/data fallback — especially important for the slope graph and scatter, which are the two least screen-reader-friendly chart types on the report.
- **No 3D, no gauges, no pie/donut** — already excluded on chart-selection grounds (§8), which also removes the accessibility issues those types carry (angle/volume encoding, poor screen-reader support).

---

## 12. Stakeholder Insights (per page)

| Page | What the stakeholder should learn | What decision this page supports |
|---|---|---|
| 1. Executive Overview | Marketing converted $229K in revenue against $19.9K in spend this year, at a 41.5% journey conversion rate | Whether marketing performance looks broadly healthy this reporting cycle |
| 2. Attribution Analysis | The same revenue is credited very differently depending on model — Paid Search/Email look like closers, Paid Social/Influencer look like openers | Which attribution model (or blended view) to standardize on for reporting |
| 3. Channel & Campaign Performance | Efficiency and raw size are not the same thing — some channels/campaigns return more per dollar than their spend size would suggest | Where to shift incremental budget among currently active channels/campaigns |
| 4. Customer Journey | Most converting journeys are short (≤3 touches) and fast (median 6 days); opening and closing channels frequently differ | Whether funnel behavior looks normal, or a segment/region needs deeper investigation |
| 5. Marketing Investment | Some channels take a bigger share of the budget than they return in attributed credit, and this credit is shared against one real revenue pool, not five | Specific channel-level budget reallocation |

---

## 13. QA Checklist

**Field/measure existence — every item below was verified against the live Phase 4 semantic model before being referenced in this spec (no invented measures, no invented fields):**

| Referenced object | Verified in |
|---|---|
| `Total Conversion Revenue`, `Total Marketing Spend`, `Total Journeys`, `Converted Journeys`, `Conversion Rate`, `Distinct Customers`, `Avg Touchpoints per Journey`, `Avg Days to Conversion`, `Median Days to Conversion`, `Revenue per Converted Customer (Linear)` | Phase 4 measure list, `attribution_customer_journeys`/`Attribution` tables |
| `First Touch Revenue`, `Last Touch Revenue`, `Linear Revenue`, `Time Decay Revenue`, `Position Based Revenue`, `Model Revenue Spread`, `ROAS First/Last/Linear/Time Decay/Position Based`, `Cost per Conversion First/Last/Linear/Time Decay/Position Based` | Phase 4 measure list, `Attribution` table |
| `DimChannel[channel]`, `Date[Date]`, `Date[YearMonth]` | Phase 4 model tables |
| `attribution_customer_journeys[channel/campaign_name/customer_segment/region/device_type/total_touchpoints/days_to_conversion/is_first_touch/is_last_touch/journey_id/journey_converted]` | Confirmed 37-column schema, Phase 4 |
| `Attribution[campaign_name]`, `Attribution[touchpoint_date]` | Confirmed 25-column schema, Phase 4 |
| Cardinalities used to justify chart/slicer choices (channels=7, campaigns=32, customer_segment=5, region=5, device_type=4, total_touchpoints max=7, days_to_conversion max=30) | Verified via live DAX query against the model during this design phase |

**Two report-layer techniques are used instead of new DAX, flagged explicitly per the brief's "no new DAX unless explicitly identified" instruction:**
1. Page 4, Visual 3 (First-Touch vs. Last-Touch channel): implicit `COUNT` of `journey_id` with a visual-level filter on `is_first_touch`/`is_last_touch` — a report-authoring filter on existing columns, not a model change.
2. Page 5, Visual 1 (Spend Share vs. Revenue Share): native "Show value as → Percent of Grand Total" display option on existing measures — a report-authoring formatting option, not a model change.

**One optional future DAX addition is identified but NOT built:** a Model Selector field parameter (§7) — flagged for explicit Phase 6 approval, not assumed.

**Design-checklist pass (per `anti-patterns.md` pre-publish workflow):**

| Check | Status |
|---|---|
| ≤7 visual groups per page | ✅ every page has 4–6 |
| No pie/donut with >5 slices anywhere | ✅ none used at all |
| No dual-axis charts | ✅ none used |
| No gauges, no 3D | ✅ none used |
| Bars start at zero | ✅ all bar/column charts |
| Every bar chart sorted by value (not alphabetical) | ✅ |
| ≤8 categorical hues per report | ✅ 3 brand hues + 1 grey context tone |
| No KPI shows an invented target or delta | ✅ confirmed in §5 |
| Max 3 slicers on any single page | ✅ Pages 3 & 4 |
| Same slicer not repeated on every page unnecessarily | ✅ only Date is global |
| Every visual traces to a stated business question | ✅ §4 |
| Every measure/field referenced exists in the model | ✅ table above |

---

**STOP condition reached.** This is a design specification only — no PBIR files, visuals, slicers, bookmarks, or themes were created, and the semantic model was not modified (read-only DAX queries were used solely to verify field/measure existence and cardinality for this spec). Waiting for approval to proceed to **Phase 6 — Build Power BI Visuals**.
