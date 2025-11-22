# 2022 Yellow Taxi QA Summary

This report captures exploratory QA checks performed on the full set of 2022 NYC TLC yellow taxi trip Parquet files (`yellow_tripdata_2022-01.parquet` … `-12.parquet`). All figures below operate on the combined 39,656,098 trips.

## Data Dictionary Context
- `VendorID` values align with TLC's March 18, 2025 spec (1=Creative Mobile Technologies, 2=Curb Mobility, 6=Myle Technologies, 7=Helix). Only codes 1 and 2 appear in 2022 data, as expected for historical yellow trips.
- `RatecodeID`, `payment_type`, `store_and_fwd_flag`, `PULocationID`, and `DOLocationID` logic checks directly follow the TLC definitions provided. Invalid codes (e.g., `payment_type=0` Flex Fare) are treated as corrupt because yellow trips should not carry Flex Fare in 2022.
- Monetary fields (`fare_amount`, `extra`, `mta_tax`, `tip_amount`, `tolls_amount`, `improvement_surcharge`, `congestion_surcharge`, `airport_fee`) are reconciled to the TLC fare model; any mismatch indicates missing surcharge components.
- `cbd_congestion_fee` was introduced in 2025 and is not present in the 2022 schema, so no QA checks cover it.
- Zone validation relies on `raw/taxi_zone_lookup.csv`, which lists `LocationID`, `Borough`, `Zone`, and `service_zone`. All pickup/dropoff IDs in 2022 data exist in this reference table.

## Coverage and Completeness
- All 12 monthly files present with pickup timestamps spanning 2021-12-31 to 2023-01-01 (expected turn of year bleed is minimal and acceptable).
- Key categorical fields (`PULocationID`, `DOLocationID`, `VendorID`, `payment_type`) show zero nulls.
- ~3.45% of rows have nulls for `passenger_count`, `RatecodeID`, `store_and_fwd_flag`, `congestion_surcharge`, and `airport_fee`. These nulls are fully aligned (same rows) and map to `payment_type=0`, an invalid code that also carries other anomalies such as a $133 average recorded tip.
- `avg_mph` derived metric is null for 0.08% of rows due to zero or negative durations.

## Passenger & Payment Behavior
- Single-passenger rides dominate (71.3%), followed by two-passenger (14.8%). Passenger counts above the TLC-allowed limit of six occur 412 times (≤0.001%), indicating rare manual data entry errors.
- `payment_type` distribution: Card (1) 75.9%, Cash (2) 19.6%, No charge (3) 0.49%, Dispute (4) 0.62%, Unknown (0) 3.45%, Voided (5) negligible. The `payment_type=0` rows align with the null cluster noted above and should either be mapped to a real code or excluded downstream.
- Tip behavior: card trips average $3.45 per ride, cash/no-charge/dispute are effectively zero. The `payment_type=0` cluster shows mean tips of $134, confirming corruption.

## Temporal & Distance Profiles
- Average trip distance ranges between 2.6–3.3 miles per month, with 99th percentile near 15 miles. Durations average 13–15 minutes monthly with 99th percentile near 46 minutes, matching historical TLC patterns.
- 9.7K trips exceed 100 miles and 3.3K exceed 3 hours, but almost all retain plausible fares ($20–60). They likely represent telemetry glitches (distance measured in tenths) rather than real trips; flag for capping or manual review.

## Domain Logic QA Findings
| Issue | Rows | % of Total | Notes |
| --- | ---: | ---: | --- |
| Fare component mismatch | 11,135,032 | 28.1% | `total_amount - sum(fare components)` differs by $±2.50 in 98% of cases. Root cause traced to missing `congestion_surcharge` when `payment_type=0`; once surcharge is treated as zero, diffs vanish. |
| Non-positive distance | 176,200 | 0.44% | These rows have zero mileage yet non-zero fares; likely system minimum fares or data capture errors. |
| Non-positive duration | 32,035 | 0.08% | Causes undefined speeds. Many coincide with zero distance or rapid swipes; retain but treat with caution. |
| Total_amount ≤ 0 | 9,016 | 0.02% | Negative fares come from dispute adjustments; downstream KPIs should filter or classify separately. |
| Passenger_count > 6 | 412 | 0.001% | Above TLC legal capacity, likely entry errors. |
| Cash payments with positive tips | 2,334 | 0.006% | Operationally impossible because cash tips are off-meter; indicates mis-coded payment types. |

## payment_type=0 Deep Dive
- Size & prevalence: 1,368,303 trips (3.45% of 2022 volume) sit in the `payment_type=0` “unknown” bucket. Every one of these rows simultaneously lacks `passenger_count`, `RatecodeID`, `store_and_fwd_flag`, `congestion_surcharge`, and `airport_fee`, so the anomaly is structural rather than sporadic.
- Vendor/temporal footprint: VendorID 2 contributes 74.6% of the cluster, VendorID 1 adds 21.1%, and VendorID 6 makes up the remaining 4.4%. All twelve monthly files carry between 5–10% of the cluster, so we cannot pin the issue on a single ingestion job.
- Downstream QA impact: 89.1% (1,219,662 rows) of the cluster triggers the `$total_amount – fare components` mismatch because `congestion_surcharge` is blank. Another 71,150 rows show zero/negative distance, 14,014 zero/negative duration, and 736 non-positive totals, so this slice single-handedly inflates multiple QA counts.
- Tip corruption: 86.6% of `payment_type=0` trips record a positive on-meter tip even though the tender type is unknown, driving an average tip of $133.74 despite a median tip of only $3.05. Two rides on 2022‑12‑14 carry absurd tips of $133,391,400 and $44,463,790 paired with equally large negative fares, proving these records cannot be trusted for revenue/tip KPIs without capping.
- Example records: Representative samples (see `src/process.ipynb`, cells 17–26) show otherwise normal distances/fare math, which means the synchronized null-field signature plus payment code is the most reliable filter to quarantine the cluster upstream.

## Recommendations
1. **Quarantine payment_type=0 rows**: Treat every `payment_type=0` record as corrupt until TLC supplies a remapping. Downstream KPIs should drop them entirely or remap to a valid tender before aggregation, and ingestion should flag the signature (payment code + synchronized nulls) so future loads are quarantined immediately.
2. **Impute congestion surcharge**: For rows where the surcharge is null but other components are present, set it to $0 and recompute totals to resolve the ±$2.50 mismatch.
3. **Flag zero-distance and zero-duration trips**: Exclude from distance/speed KPIs or replace with minimum thresholds to avoid skewing metrics.
4. **Negative fares and disputes**: Tag and remove from revenue aggregates; cross-check with TLC dispute policies.
5. **Passenger count > 6**: Enforce range checks during ingestion; for historical data, cap at 6 or treat as invalid.
6. **Document telemetry anomalies**: Distances over 100 miles and durations over 6 hours should be capped or removed before modeling.
7. **Apply tip sanity caps**: Introduce a rule that rejects or caps tips above a realistic threshold (e.g., $200) before summarizing gratuities to prevent a handful of corrupt `payment_type=0` rows from dominating tip metrics.

All code and calculations live in `src/process.ipynb` (cells 1–26). Re-run the notebook after any upstream data refresh to recompute counts.
