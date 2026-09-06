# DATA CLEANING REPORT — v2 (Revised Duplicate Treatment)
**Phase 2 Revision — Attribution Customer Journeys**
Source: `data/raw/attribution_customer_journeys.csv` (untouched)
Previous cleaned dataset: `data/processed/attribution_customer_journeys_clean.csv` (preserved, not overwritten)
New output: `data/processed/attribution_customer_journeys_clean_v2.csv`
Duplicate audit: `data/processed/duplicate_audit_v2.csv`

---


## Duplicate Investigation — what the data actually showed

I grouped the raw data by `(journey_id, touchpoint_position)` and examined every group with more than one record, comparing 18 substantive fields (channel, category, campaign, UTM, conversion status, revenue, first/last-touch flags, etc. — excluding only tracking-level fields like `touchpoint_id`/`session_id`/timestamp, which are expected to differ even for a genuine same-instant double-fire).

| Finding | Result |
|---|---|
| Duplicate groups (position appears >1 time in a journey) | **99**, all of size exactly 2 (no groups of 3+) |
| Total rows involved | 198 |
| Groups where **both** copies are flagged `is_duplicate_flag=1` | **99 of 99 (100%)** — there is no case with one flagged + one clean copy |
| Groups where the two copies are **substantively identical** (same channel, category, campaign, conversion status, revenue — differ only in tracking fields) | **99 of 99 (100%)** |
| Groups with a genuine field conflict (different channel, different conversion status, different revenue, etc.) | **0** |

This confirms the README's own description: tracking-pixel double-fires, same user action recorded twice, not two different events.

## Canonical Record Selection Rule (documented, evidence-based)

Applied in priority order, per duplicate group:

1. **If exactly one copy has `is_duplicate_flag = 0`**, retain it — the tracking system's own signal for which copy is canonical. *(Did not occur in this dataset — see below.)*
2. **If all copies are flagged (`is_duplicate_flag = 1` on every copy)** — the actual pattern found in all 99 groups — first verify the copies are substantively identical (18-field comparison above). If identical, retain the copy with the **lowest `touchpoint_id`** as a deterministic, documented tie-break. Since the discarded copy carries identical substantive data, no information is lost by this choice.
3. **If copies conflict on any substantive field**, do not choose — flag the entire group as `FLAGGED_CONFLICT` and retain all copies pending manual review, rather than silently picking one.

Result: **0 groups required flagging.** All 99 were resolved under rule 2.

The full group-by-group record is in `duplicate_audit_v2.csv` — columns: `duplicate_group_id, journey_id, touchpoint_position, num_duplicate_records, records_examined, record_retained, reason_retained, conflicting_fields, status`. Every row's retention decision is individually documented.

---

## Conversion Protection Verification (critical check)

Run against every one of the 1,454 converted journeys:

| Check | Result |
|---|---|
| Exactly one last-touch record remains | ✅ 0 failures |
| Last-touch position equals `total_touchpoints` | ✅ 0 failures |
| Exactly one touchpoint carries `converted=1` | ✅ 0 failures |
| `journey_order_value` > 0 | ✅ 0 failures |
| Final channel is identifiable | ✅ 0 failures |

**CONVERSION PROTECTION: PASSED.** The 10 journeys that lost their converting touchpoint in v1 are now fully intact in v2.

---

## Journey Integrity (re-run in full)

| Check | Result |
|---|---|
| `touchpoint_position` starts at 1 | 0 journeys fail |
| `touchpoint_position` ends at `total_touchpoints` | 0 journeys fail |
| No gaps in position sequence | 0 journeys fail |
| No duplicate positions remain | 0 journeys fail |
| `is_first_touch` sums to exactly 1 per journey | 0 journeys fail |
| `is_last_touch` sums to exactly 1 per journey | 0 journeys fail |
| Actual touch count matches `total_touchpoints` | 0 journeys fail |

**Journeys with fully complete, gap-free touchpoint sequences: 3,500 of 3,500 (100.00%)** — up from 0% clean under v1's approach (94 of 99 were still broken).

---

## Revenue Integrity (journey grain — not summed across repeated rows)

| Metric | Value |
|---|---|
| Total revenue, journey grain, **before** cleaning | $229,294.41 |
| Total revenue, journey grain, **after** v2 cleaning | $229,294.41 |
| **Difference** | **$0.00** |
| Journeys where summed `order_value_usd` ≠ `journey_order_value` | 0 |
| Converted journeys before | 1,454 |
| Converted journeys after | 1,454 |

Deduplication changed **zero dollars** of true conversion revenue — exactly as required. (v1, by contrast, lost $1,783.66 and 3 converted journeys' worth of tracking due to the flawed removal logic.)

---

## Attribution Model Readiness

| Model | Requirement | Ready? |
|---|---|---|
| First Touch | Exactly one `is_first_touch=1` per journey at position 1 | ✅ Yes |
| Last Touch | Exactly one `is_last_touch=1` per journey, matching the converting touchpoint | ✅ Yes |
| Linear | Complete, gap-free touchpoint set per journey (equal split needs every real touchpoint) | ✅ Yes (3,500/3,500 complete) |
| Time Decay | Same as Linear, plus a valid timestamp on every touchpoint | ✅ Yes (0 invalid dates) |
| Position-Based | Complete sequences plus reliable first/last-touch flags | ✅ Yes |

**All five models are ready to build in Phase 3.** No attribution calculation was performed in this phase.

---

## Missing Fields — required for attribution?

| Field | Missing | Used by any of the 5 attribution models? | Verdict |
|---|---|---|---|
| `device_type` | 169 rows (1.51%) | No — all 5 models key on `touchpoint_position`, `is_first_touch`/`is_last_touch`, `touchpoint_date`, and `channel`. `device_type` is a segmentation dimension only. | **Non-blocking data-quality limitation.** Not filled, per instructions. |
| `touchpoint_hour` | 46 rows (0.41%) | No — used only for hour-of-day breakdowns, not by any model. | **Non-blocking data-quality limitation.** Not filled, per instructions. |

---

## Final Validation Summary (exact figures requested)

| Metric | Value |
|---|---|
| Original rows | 11,292 |
| Rows flagged as duplicates (`is_duplicate_flag=1`) | 198 |
| Duplicate groups (`journey_id`+`touchpoint_position`, count>1) | 99 |
| Rows removed (non-canonical duplicates) | 99 |
| Rows retained from duplicate groups (canonical) | 99 |
| Final rows | **11,193** |
| Journeys before | 3,500 |
| Journeys after | **3,500** (all preserved) |
| Journeys with complete touchpoint sequences | **3,500 (100.00%)** |
| Converted journeys | **1,454** |
| Converted journeys with valid last touch | **1,454 (100%)** |
| Revenue before | $229,294.41 |
| Revenue after | $229,294.41 |
| Revenue difference | **$0.00** |
| First-touch readiness | ✅ Ready |
| Last-touch readiness | ✅ Ready |
| Linear readiness | ✅ Ready |
| Time-decay readiness | ✅ Ready |
| Position-based readiness | ✅ Ready |

---

## Ambiguous / Conflicting Groups

**None.** All 99 duplicate groups were substantively identical and resolved deterministically under a documented rule. Zero groups required flagging for manual review. This is a real finding, not an assumption — every group was individually compared across 18 substantive fields before any resolution was applied (see `duplicate_audit_v2.csv`).

---

## Still outstanding (non-blocking, unchanged)

- `campaign_name` blanks (208 rows) use `"(No Campaign)"` in this file; the project's own notebook uses `"Unknown / Direct"`. Your call whether to reconcile.
- `device_type` (169 rows) and `touchpoint_hour` (46 rows) missingness — confirmed non-blocking for attribution, left unfilled.

---

**STOP condition reached.** No attribution models, DAX, or Power BI visuals were built. `attribution_customer_journeys_clean_v2.csv` and the previous `attribution_customer_journeys_clean.csv` both exist for auditability. Waiting for approval to proceed to Phase 3 — Attribution Model Construction.
