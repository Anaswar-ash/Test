## 1. Raw Data Profile (Pre-Cleaning)

**Shape:** 30,200 rows × 11 columns

**Column Types**
- Text/categorical: `tradets`, `sym`, `event_type`, `order_type`, `side`, `order_id`, `endtag`
- Numeric: `price`, `quantity`, `client_tag`, `inception_pnl`

**Missing Values Identified**

| Column | Missing |
|---|---|
| `tradets` | 176 |
| `side` | 42 |
| `price` | 300 |
| `client_tag` | 273 |
| `endtag` | 206 |
| All others | 0 |

**Numeric Field Interpretation**

- `price`: Negative values present (min = −24,577.5) — impossible as real market prices.
  Extreme max of 9,999,999 is almost certainly a sentinel/error code.
  Typical values (median 158.2, Q3 4,995.25) look like realistic FX/XAUUSD/NAS100 levels.
- `quantity`: Negative values present (min = −49) — nonsensical as volume since
  direction is already encoded in `side`. Max of 49.8M indicates extreme outliers
  or corrupted fields.
- `client_tag`: Fixed range (21,389 to 73,106) with clean quartiles — no obvious errors.
- `inception_pnl`: Range −74.6 to +78.6, roughly symmetric — no obvious errors.

---

## 2. Cleaning Summary

**Rows removed by symbol-specific price bounds:** 81  
**Exact duplicates dropped:** 197  
**Final cleaned dataset:** 29,330 rows

**Post-Cleaning Numeric Profile**

| Metric | price | quantity | client_tag | inception_pnl | ts_seconds |
|---|---|---|---|---|---|
| count | 29,330 | 29,330 | 29,330 | 29,330 | 29,330 |
| min | 1.1475 | 0.0 | −1.0 | −74.577 | 0.0 |
| median | 158.23 | 37.0 | 51,842 | 8.72 | 1,801.75 |
| max | 24,623.5 | 49,852,200 | 73,106 | 78.579 | 3,599.7 |

**Price ranges by instrument after cleaning**

| Instrument | Min | Max |
|---|---|---|
| EURUSD | 1.1475 | 1.1514 |
| GBPUSD | 1.3270 | 1.3304 |
| USDJPY | 158.185 | 159.145 |
| XAUUSD | 4,987.60 | 5,010.70 |
| NAS100 | 24,456.00 | 24,623.50 |

**Dataset Segmentation**

| Dataset | Rows | Description |
|---|---|---|
| `df_clean` | 29,330 | Full cleaned event stream |
| `df_dir` | 29,248 | Directional trades only (side ∈ {buy, sell}) |
| Excluded | 82 | Non-directional / null side (market-making) |

---

## 3. Analytic Findings

### 3.1 Market Activity
- EURUSD dominates by event count and total traded volume
- Trade share is strikingly consistent at **~33%** across all five instruments —
  roughly one in three messages results in an actual execution regardless of instrument,
  suggesting a systematic and disciplined order submission pattern
- Directional flow is almost perfectly balanced at **50/50 buy vs sell**,
  consistent with deliberate hedging or risk-neutral systematic execution
- The 50/50 split is computed on `df_dir` — the subset of `df_clean` restricted to
  `side ∈ {buy, sell}` — excluding the 82 non-directional and null-side rows so the
  ratio is not distorted by unclassified flow

### 3.2 Execution Efficiency — Fill Rate
- Fill rates exceed **95%** on every instrument — indicating aggressive IOC-style execution
- EURUSD fill rate **101.3%** — partial fills generating multiple TRADE events per NEW order
- USDJPY lowest at **95.6%** — marginally thinner liquidity or wider spreads

### 3.3 Time Profile
- Cancel-to-trade ratio **never crosses 1.0** in any single minute window
- Session average **0.60x** — executions consistently outnumber cancellations throughout
- Session peak at **minute 39 (0.76x)** — flagged for abnormal activity investigation

### 3.4 Notional Value
- NAS100 dominates economic exposure — median executed trade **~$688K**, max **$26.85M**
- XAUUSD second — median notional **~$140K** per trade
- FX pairs (EURUSD, GBPUSD) median **~$50** — fractional lot sizes, not standard FX convention
- All top 10 largest trades by notional are NAS100

### 3.5 Strategy PnL Leaderboard
- **26 strategies** tracked across the session
- **16 winners (61.5%)**, **10 losers (38.5%)**
- Total session PnL: **$233.48**
- Most profitable: **66214_ld_kjdfiefiriig (+$39.05)**
- Heaviest loss: **47362_oijuhygtr45 (−$22.41)**
- Top strategy accounts for only **12.4% of total winning PnL** —
  profits are broadly distributed, not concentrated in a single name
- **$10.58 unattributed** under `UNKNOWN_ENDTAG` — requires re-tagging in production

---

## 4. Non-Obvious Insights

- **Execution discipline:** The cancel-to-trade ratio never exceeds 0.76x in any minute
  and averages 0.60x — this consistency is invisible from aggregate counts alone and
  only emerges from the minute-level time profile
- **PnL vs notional disconnect:** Total session PnL is only $233.48 against tens of
  millions in NAS100 notional exposure — implying tight risk limits, delta-neutral
  positioning, or a normalised return metric rather than raw dollar PnL
- **Profit breadth:** The top strategy holds only 12.4% of total winning PnL —
  the book is not dependent on any single strategy, which is a structural health signal
  that would not be visible from total PnL alone