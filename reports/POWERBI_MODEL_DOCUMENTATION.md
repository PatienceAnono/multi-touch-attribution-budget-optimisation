# POWER BI MODEL DOCUMENTATION
**Phase 4 — Power BI Data Model + DAX**

Model built directly in the open Power BI Desktop session (untitled, unsaved at time of writing — no `.pbix`/`.pbip` exists yet). No visuals, slicers, themes, bookmarks, or navigation were created in this phase. The separate `Ecommerce_Dashboard` project/file was not touched.

---

## 1. Tables

| Table | Rows | Columns | Grain | Source |
|---|---|---|---|---|
| `attribution_customer_journeys` | 11,193 | 37 | One touchpoint (all journeys, converted or not) | `data/processed/attribution_customer_journeys_clean_v2.csv` |
| `Attribution` | 4,658 | 25 | One touchpoint within one **converted** journey, with all 5 models' weights/revenue | `data/processed/attribution_output.csv` |
| `Date` | 366 | 10 | One calendar day, 2024-01-01 to 2024-12-31 | Calculated table (`CALENDAR()` + `ADDCOLUMNS`), marked as the official Date Table |
| `DimChannel` | 7 | 1 | One row per channel | Calculated table (`DISTINCT` of `attribution_customer_journeys[channel]`) |

**Correction made to a pre-existing table:** `attribution_customer_journeys` was already loaded in the session before this phase began, but its M-query pointed at the **raw** file (`data/raw/attribution_customer_journeys.csv`, 11,292 rows, none of the Phase 2 fixes applied). Per user confirmation, its partition was repointed to the validated `attribution_customer_journeys_clean_v2.csv` and refreshed — row count moved from 11,292 (raw) to 11,193 (v2, canonical-deduplicated). No other pre-existing objects required correction.

## 2. Relationships

| From | To | Cardinality | Active | Purpose |
|---|---|---|---|---|
| `attribution_customer_journeys[touchpoint_date]` | `Date[Date]` | Many:1 | Yes | Primary time axis for the full journey population |
| `attribution_customer_journeys[conversion_date]` | `Date[Date]` | Many:1 | No (inactive) | Available via `USERELATIONSHIP()` for conversion-date-based time intelligence |
| `Attribution[touchpoint_date]` | `Date[Date]` | Many:1 | Yes | Primary time axis for the attribution output |
| `attribution_customer_journeys[channel]` | `DimChannel[channel]` | Many:1 | Yes | Channel bridge between the two fact tables (see §3) |
| `Attribution[channel]` | `DimChannel[channel]` | Many:1 | Yes | Channel bridge between the two fact tables (see §3) |

`attribution_customer_journeys` and `Attribution` have **no direct relationship to each other** — they are bridged only through the shared dimensions (`Date`, `DimChannel`). This is deliberate: the two tables have different grains (all journeys vs. converted-journey touchpoints only) and joining them directly on `touchpoint_id`/`journey_id` would create ambiguous many-to-many filtering.

## 3. Star-Schema Assessment (Step 3)

**Decision: minimal snowflake, not a full star schema.** Built only what a real, observed need justified:

- **`Date` — built.** Required by the task; also replaces Power BI's Auto Date/Time (see §4).
- **`DimChannel` — built, for an evidence-based reason found during validation, not planned upfront.** Initial testing showed that channel-level ROAS and Cost-per-Conversion measures were silently wrong: `Total Marketing Spend` (from `attribution_customer_journeys`) and model revenue (from `Attribution`) live in two unrelated tables, so grouping by `Attribution[channel]` did not filter `attribution_customer_journeys` at all — every channel showed the same grand-total spend in the denominator. A 7-row `DimChannel` bridge, related to both fact tables, fixed this; verified below in §6.
- **`DimCampaign`, `DimRegion`, `DimSegment`, `DimCustomer` — deliberately NOT built.** No measure in this phase mixes `attribution_customer_journeys` and `Attribution` on campaign, region, segment, or customer the way cost/ROAS mixed on channel — every Campaign Performance and Customer Analysis measure resolves correctly from a single table's own column. Cost/ROI was validated by the Python phase (`ATTRIBUTION_VALIDATION_REPORT.md`) at **channel** grain only, not campaign grain, so there is no established, validated target to reconcile a campaign-cost bridge against. Building these dimensions now would add relationship complexity with no measure that needs them — reconsider only if a future phase requires cross-table filtering on one of these fields.

**Reporting implication for Phase 5:** any visual mixing spend and revenue by channel must filter through `DimChannel[channel]`, not `Attribution[channel]` or `attribution_customer_journeys[channel]` directly, to get correct cross-table results.

## 4. Date Dimension (Step 4)

A calculated `Date` table (`CALENDAR(DATE(2024,1,1), DATE(2024,12,31))`) replaces reliance on Power BI's Auto Date/Time. It was marked as the model's official Date Table via `Mark as Date Table` on the `Date` column. The three auto-generated hidden `LocalDateTable_*`/`DateTableTemplate_*` objects and their relationships (previously attached to `touchpoint_date`, `touchpoint_datetime`, `conversion_date`) were deleted first — deleting a relationship whose column carries an Auto Date/Time `Variation` requires deleting the underlying hidden date table as well, or the column becomes unreadable until the hidden table is removed (encountered and resolved during this build).

Columns: `Date`, `Year`, `MonthNumber`, `MonthName` (sorted by `MonthNumber`), `YearMonth`, `Quarter`, `WeekNumber`, `DayName` (sorted by a hidden `DayOfWeekNumber`), `IsWeekend`.

`touchpoint_datetime` was **not** related to `Date` — it's a finer-grain timestamp with no analytical use identified beyond the day-level `touchpoint_date`, and Power BI only permits one active relationship per table pair; adding a second inactive one for a field nothing currently uses would be unused complexity.

## 5. Measures (43 total, 8 categories, all as DAX measures — no new calculated columns added to the CSV-sourced tables)

Every attribution number is a direct `SUM()`/reference over the Python-computed columns in `Attribution` (`first_touch_revenue`, `last_touch_weight`, etc.) — **no attribution logic (weights, decay, position rules) is recalculated in DAX.** DAX only aggregates, compares, and reconciles the numbers Python already validated.

| Category | Measures |
|---|---|
| **Core Business** | Total Journeys · Converted Journeys · Conversion Rate · Total Touchpoints · Avg Touchpoints per Journey · Total Marketing Spend · Total Conversion Revenue |
| **Attribution Models** | First Touch Revenue · Last Touch Revenue · Linear Revenue · Time Decay Revenue · Position Based Revenue |
| **Model Comparison** | Max Model Revenue · Min Model Revenue · Model Revenue Spread |
| **Channel Performance** | ROAS First/Last/Linear/Time Decay/Position Based (5) · Cost per Conversion First/Last/Linear/Time Decay/Position Based (5) |
| **Campaign Performance** | Campaign Count · Journeys with No Campaign |
| **Customer Analysis** | Distinct Customers · Converted Customers · Avg Journeys per Customer · Revenue per Converted Customer (Linear) |
| **Time Analysis** | Avg Days to Conversion · Median Days to Conversion |
| **QA and Reconciliation** | QA Diff (First/Last/Linear/Time Decay/Position Based vs. Total Conversion Revenue) (5) · QA Weight Sum Check (First/Last/Linear/Time Decay/Position Based — each should equal 0, since weights must sum to 1 per journey) (5) |

Cost-per-Conversion uses each model's **fractional weight sum** as a credit-weighted conversion count (`Total Marketing Spend ÷ SUM(model_weight)`), matching the Python methodology (`ATTRIBUTION_METHODOLOGY.md` §12) rather than an inflated "any journey that touched this channel" count. ROAS and Cost-per-Conversion correctly return **blank** (not $0 or an error) for Direct and SEO/Organic, whose spend is genuinely $0 — consistent with Python's "undefined, not fabricated" treatment.

## 6. Validation Results (Step 13)

All figures below were queried directly from the live Power BI model via DAX and compared to the Python Phase 3 outputs.

| Check | Power BI | Python source of truth | Match |
|---|---|---|---|
| Total Journeys | 3,500 | 3,500 | ✅ |
| Converted Journeys | 1,454 | 1,454 | ✅ |
| Total Touchpoints (journey population) | 11,193 | 11,193 (v2) | ✅ |
| Total Marketing Spend | $19,901.84 | $19,901.84 (sum of channel_cost_roi.csv) | ✅ |
| Total Conversion Revenue | $229,294.41 | $229,294.41 | ✅ |
| First Touch Revenue | $229,294.41 | $229,294.41 | ✅ |
| Last Touch Revenue | $229,294.41 | $229,294.41 | ✅ |
| Linear Revenue | $229,294.41 | $229,294.41 | ✅ |
| Time Decay Revenue | $229,294.41 | $229,294.41 | ✅ |
| Position Based Revenue | $229,294.41 | $229,294.41 | ✅ |
| QA Diff, all 5 models | ~1×10⁻¹⁰ (floating-point noise) | $0.00 (same noise level) | ✅ |
| QA Weight Sum Check, all 5 models | 0 (±4.5×10⁻¹³ noise) | Weights sum to 1.0000000000 per journey | ✅ |
| Avg Days to Conversion | 7.4154 | 7.42 | ✅ |
| Median Days to Conversion | 6.0 | 6.0 | ✅ |
| Channel spend (Paid Search / Influencer / Paid Social / Display / Email / Direct / SEO) | $8,528.47 / $6,549.29 / $4,188.60 / $367.82 / $267.66 / $0.00 / $0.00 | identical | ✅ |
| Channel revenue by model (all 7 channels × 5 models, spot-checked First/Last/Linear) | identical to the cent | identical | ✅ |

**No discrepancy of any kind was found or needed adjusting to force a match** — every number reconciled on the first correctly-modeled query, aside from the channel-cost bridging issue described in §3, which was a model-structure gap (fixed with `DimChannel`), not a data or DAX-logic error.

## 7. Known Limitations

- Cost/ROAS/Cost-per-Conversion is only reliable at **channel** grain (via `DimChannel`). Campaign-, region-, segment-, or device-level cost efficiency is not currently computable without building an equivalent bridge dimension — out of scope for this phase since Python's own validated cost analysis (§Phase 3) was channel-grain only.
- `conversion_date`'s relationship to `Date` is inactive by default (Power BI allows only one active relationship per table pair); any conversion-date-based time intelligence must use `USERELATIONSHIP(attribution_customer_journeys[conversion_date], 'Date'[Date])` inside `CALCULATE`.
- The `Date` table spans exactly 2024-01-01 to 2024-12-31, matching the observed data range (`touchpoint_date`: 2024-01-01 to 2024-12-19; `conversion_date`: 2024-01-03 to 2024-12-22) plus the rest of the calendar year for clean month/quarter totals — it is not open-ended and would need extending if new data outside 2024 arrives.
