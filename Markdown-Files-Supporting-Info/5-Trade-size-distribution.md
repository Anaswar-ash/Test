## 5. Trade Size Distribution

Markdown – 5.1 Goal  
Markdown – 5.2 Method  
Code – 5.x Whale summary + concentration table  
Markdown – 5.3 Results

### 5.1 Goal

Building on the notional value baseline and visual trade‑size distributions established in Section 2.5, this section formally characterises the distribution tails for each instrument. The objective is to identify unusually large trades (“whales”) using a transparent, instrument‑specific statistical rule, and to quantify how much of the total session risk is concentrated in those tail events rather than in the body of the distribution.

---

### 5.2 Method

This analysis does not redefine metrics or recreate the boxplots from Section 2.5. Instead, it directly leverages the existing `dftrades` DataFrame of executed trades and the `notional` column already defined as:

- `notional = price × quantity`, capturing the true economic size of each fill.

The tail behaviour and concentration are summarised in a single per‑instrument table using the following definitions:

- **Outlier definition (Whale rule).**  
  An “unusually large” trade is any execution where the notional size is greater than or equal to the 99th percentile of the notional distribution for that specific instrument. The 99th percentile is computed separately per symbol (`sym`) so that thresholds scale to the typical size profile of each book (NAS100 vs XAUUSD vs FX pairs).

- **Justification.**  
  A percentile‑based rule is robust across asset classes, automatically adapts to different price levels and trading conventions, and avoids arbitrary fixed monetary cut‑offs. Focusing on the top 1% of trades by size aligns with how risk and surveillance teams typically isolate tail events for review.

- **Whale concentration metric.**  
  For each instrument, the total notional traded over the session is compared with the notional contributed by whale trades (those at or above the 99th percentile). The resulting ratio is expressed as a **Whale Concentration %**, showing what fraction of total risk transfer is carried by the largest 1% of trades.

The accompanying Python cell constructs a single consolidated summary table by instrument, reporting trade counts, central and tail quantiles (median, 90th, 99th percentile), maximum trade size, and Whale Concentration %, sorted so that instruments with the most concentrated tail risk appear at the top.

---

### 5.3 Visual Check of Whale Thresholds (Illustrative)

To complement the tabular Whale Concentration view, I include a single static trade‑size histogram for NAS100. The chart overlays the 99th‑percentile threshold on the empirical trade‑size distribution, making the separation between “normal” flow and whale trades visually explicit in one of the most heavy‑tailed books.

### 5.4 Results: Institutional Blocks vs Retail‑Style Flow

The percentile and concentration analysis confirms two distinct trade‑size regimes in this session. In NAS100 and XAUUSD, a “barbell” regime dominates: as already visible in the Section 2.5 notional boxplot, their trade‑size distributions are extremely right‑skewed, with 99th‑percentile notionals that are multiples of the median. The Whale Concentration % for these instruments is high, indicating that the top 1% of trades account for a disproportionately large share of total notional. Risk in these books is therefore “lumpy” and driven by isolated institutional‑scale block trades rather than by the average ticket.

For EURUSD, GBPUSD, and USDJPY, a much more “granular” regime prevails. The 99th percentile of notional remains relatively close to the median, and the Whale Concentration % is markedly lower, meaning that the top 1% of trades carry only a modest fraction of total volume. Risk in these FX pairs is built through the high‑frequency accumulation of many small, likely fractional‑lot trades rather than through occasional very large prints.

From a surveillance and intraday limit perspective, this implies that tail monitoring is most critical for NAS100 and XAUUSD, where a handful of whale trades can move substantial risk and warrant focused review. For the FX instruments, monitoring should focus more on aggregate exposure over time and on strategy‑level behaviour than on individual trade sizes, since no single trade materially dominates the risk budget.