# Data Quality Tracker for `df_clean`

## 1. Overview

`df_clean` is the cleaned event stream derived from the raw `codingtestdata.csv` file. It is used as the **single source of truth** for all subsequent analytics (Sections 2–4), including time profiling, abnormal activity detection, and spread vs price dynamics.

Key characteristics:
- 28,940 cleaned events after filtering invalid time, price, quantity, and event types.
- Instruments: EURUSD, GBPUSD, USDJPY, XAUUSD, NAS100.
- All numeric fields coerced to numeric types; invalid entries converted to `NaN` and dropped where necessary.
- Time normalised to a numeric `tsseconds` field, representing seconds from session start (MMSS.s → float seconds).

The sections below document **known residual limitations** in `df_clean` that are either intentional (by design) or unavoidable (due to synthetic test data constraints).

## 2. Residual Data Issues

### 2.1 Order ID Collisions (Synthetic Lifecycle Artefacts)

- 27 distinct `orderid` values appear in multiple instruments and/or opposite sides, so they cannot represent a single real order lifecycle.
- These collisions cause **false multi-fill signals** in naive per-order analytics and explain the >100% EURUSD fill rate seen before the revised analysis.
- They are treated as **synthetic data artefacts** rather than genuine partial fills.

Implications:
- `orderid` cannot be used for deterministic life-cycle reconstruction, partial-fill logic, or matching-engine style order books.
- Any order-level surveillance (wash trades, self-matching, parent/child analysis) must **not rely on `orderid` alone** and should be framed as structural examples for this test session only.

### 2.2 Time Precision and tsseconds/tsint

- The raw `tradets` field is a string with MMSS.s format and limited to ~0.1 second precision.
- A numeric time field `tsseconds` is derived as a float (e.g. `1802.4`), and all time-based analytics use this field.
- For some analyses (e.g. minute buckets, 1-second L1 proxy), `tsseconds` is downcast to integer seconds: `tsint = tsseconds.astype(int)` or `minute = tsseconds // 60`.

Implications:
- Multiple events within the same second but different 100ms offsets are **collapsed** when grouping by integer seconds.
- Any condition phrased as "same timestamp" is effectively a **1-second window**, not a tick-perfect equality.
- For this single-session, exploratory test, second-level granularity is acceptable, but it would be too coarse for full HFT microstructure reconstruction.

### 2.3 Statistical L1 Proxy for Spreads (Not a Deterministic LOB)

- Due to order ID collisions (Section 2.1), a deterministic Limit Order Book (matching NEW/MODIFY/CANCEL per order) cannot be safely reconstructed.
- Instead, Section 4 uses a **Statistical L1 Proxy**:
  - Filter to events with `event_type` in {NEW, MODIFY, TRADE}.
  - For each symbol and second, compute:
    - `best_bid` = max buy price.
    - `best_ask` = min sell price.
  - Forward-fill best bid/ask per symbol to simulate a continuous top-of-book.
- Negative spreads (crossed markets) are treated as proxy artefacts and filtered out before analysis.

Implications:
- The resulting spreads and mid-prices are **approximate per-second snapshots**, not the result of a real matching engine.
- Spreads should be interpreted as **upper-bound indications of liquidity conditions**, suitable for regime analysis and spread-vs-move statistics, but **not** for tick-perfect market microstructure work.

### 2.4 Synthetic / Non-standard Quantity Scale

- Quantity values have been cleaned (absolute value, lower bound of 1 for NEW/TRADE, instrument-specific upper bounds), but they do **not** align with standard FX or futures lot conventions.
- FX instruments in particular show small, irregular quantities that look like arbitrary units rather than standard contract sizes.

Implications:
- `quantity` is reliable as a **relative size measure inside this dataset**, but not as a direct proxy for real-world lot sizing.
- Economic size is captured via `notional = price * quantity`, which is used for cross-instrument comparisons (e.g. top-10 trades, notional distributions).
- The analysis intentionally **does not interpret 1 unit as 1 lot** or attempt to back out leverage or margin.

### 2.5 UNKNOWN Client and Strategy Tags

- The raw data contains missing `clienttag` and `endtag` values.
- In `df_clean`, missing values are filled with neutral labels:
  - `clienttag = 'UNKNOWNCLIENT'` where original values were missing.
  - `endtag = 'UNKNOWNENDTAG'` where original values were missing.

Implications:
- All events remain in group-level aggregations, but a subset of activity is rolled into an **unattributed UNKNOWN bucket**.
- Any per-strategy or per-client analytics that include UNKNOWN are partially opaque and should be described as including **unattributed flow**.
- For production use, these UNKNOWN labels would trigger a separate data-quality investigation with upstream systems.

### 2.6 Tick-Size Granularity Not Enforced

- Price filters enforce **instrument-specific ranges** (e.g. EURUSD around 1.15, GBPUSD around 1.33, USDJPY around 159, XAUUSD around 5,000, NAS100 around 24,500), removing obviously corrupt or sentinel prices.
- The cleaning deliberately **does not** enforce venue-specific tick sizes (e.g. 0.0001 for EURUSD/GBPUSD, 0.01 for USDJPY), to keep the focus on range-based sanity checks.[web:111]

Implications:
- All prices in `df_clean` are within reasonable ranges for their instruments, but some ticks may still be **off-grid** relative to true venue tick sizes.
- For this exercise, range-cleaned prices are sufficient for fill rates, notional analysis, and spread regimes.
- In production, an additional pass would validate tick-size conformity per venue and symbol.

### 2.7 Single-Session Scope

- `df_clean` covers a **single 60-minute trading session**.
- Many structural screens (wash trades, strategy dominance, volume spikes, spread-vs-move regressions) are demonstrated on this session only.

Implications:
- All conclusions are framed as **"within this one-hour test session"**, not as long-term behaviour of strategies or instruments.
- Production-grade surveillance and model calibration would repeat the same logic across **30+ days** of data and incorporate position context, repeated patterns, and client-level monitoring.

## 3. Summary

`df_clean` is robust enough to support the analytics requested in the exercise, but it remains subject to several **known structural limitations**: order ID collisions, approximate L1 spreads, synthetic quantity scales, UNKNOWN tags, potential off-tick prices, and single-session scope.

These limitations are explicitly acknowledged and factored into the design of each section:
- Order-level analyses avoid hard dependence on `orderid` and are presented as structural examples.
- All spread and price-dynamics work relies on a per-second Statistical L1 Proxy rather than a deterministic order book.
- Size-based metrics use notional value rather than interpreting quantity as real-world lots.
- Strategy and client-level insights are qualified where UNKNOWN tags are present.

This tracker is intended as a living document to accompany the notebook, making explicit where the data is strong, where it is synthetic, and how those constraints shape the analytics.
