## 4. Spread vs Price Dynamics

### 4.1 Overview and Interpretation Framework

This section investigates how market liquidity, proxied by the bid-ask spread, relates to short-horizon price movements for each instrument in the session.  
The central question is whether trading in a wide-spread regime exposes a participant to a more volatile or directional tape than trading in a tight-spread regime.

When commenting on results, the analysis will focus on four key questions:

1. Do wider spreads systematically coincide with larger 5-second price moves?  
2. Is the effect consistent across all instruments, or concentrated in more volatile products like NAS100 or XAUUSD?  
3. Do wide-spread regimes show directional bias, or mostly increased volatility magnitude?  
4. Does the Minute 39 cancel surge align with wider-than-normal spreads?

Because the analysis is based on a one-hour synthetic session and an approximate L1 proxy, all conclusions are framed as **session-specific structural evidence**, not production-calibrated impact estimates.

---

### 4.2 Data Integrity and Statistical L1 Proxy

As established earlier in the notebook, this dataset contains synthetic order ID collisions, where the same `order_id` appears across different instruments and/or opposite sides.  
That makes deterministic order-book reconstruction unreliable, because a true `NEW`–`MODIFY`–`CANCEL` life cycle can no longer be traced safely from `order_id` alone.

To handle this, Section 4 deliberately avoids deterministic book-building and instead constructs a Statistical L1 Proxy:

- Restrict to `NEW`, `MODIFY`, and `TRADE` events only.  
- For each symbol and second, take:
  - the highest buy price as **Best Bid**,  
  - the lowest sell price as **Best Ask**.  
- Forward-fill the resulting bid and ask series by symbol to approximate a continuous top-of-book.  
- Drop intervals where no two-sided market exists yet.  
- Filter out negative spreads as crossed-market proxy artefacts.

This proxy is not a perfect Limit Order Book, but it is the most robust way to approximate liquidity conditions given the known structural issues in the data.

---

### 4.3 Feature Engineering and Methods

Using the proxy book, the following features are defined for each instrument:

**1. Mid-Price ($M_t$)** The reference price at time $t$:  
$$M_t = \frac{\text{Best\_Bid}_t + \text{Best\_Ask}_t}{2}$$

**2. Relative Spread ($\text{Spread\_bps}_t$)** Spread normalised to basis points for cross-asset comparison:  
$$\text{Spread\_bps}_t = \left( \frac{\text{Best\_Ask}_t - \text{Best\_Bid}_t}{M_t} \right) \times 10,000$$

**3. 5-Second Forward Return ($Ret_{5s,t}$)** Directional mid-price change over the next 5 seconds:  
$$Ret_{5s,t} = \left( \frac{M_{t+5} - M_t}{M_t} \right) \times 10,000$$

**4. 5-Second Absolute Return ($Abs\_Ret_{5s,t}$)** The magnitude of the move, regardless of direction:  
$$Abs\_Ret_{5s,t} = |Ret_{5s,t}|$$

These features convert the raw event stream into a comparable liquidity-versus-volatility framework across NAS100, XAUUSD, EURUSD, GBPUSD, and USDJPY.

#### 4.3.1 Spread Regime Analysis

For each instrument, observations are grouped into four spread quartiles to provide a direct non-linear comparison between tight and wide conditions:

| Regime | Definition | Output Metrics Computed |
| :--- | :--- | :--- |
| **Q1_Tight** | Tightest 25% of spreads | $\bullet$ Average spread (bps) |
| **Q2_Normal** | Lower half of normal conditions | $\bullet$ Avg absolute 5s move (bps) |
| **Q3_Normal** | Upper half of normal conditions | $\bullet$ Avg directional 5s return (bps) |
| **Q4_Wide** | Widest 25% of spreads | $\bullet$ Number of observations |

#### 4.3.2 OLS Regression

To summarise the relationship with a single coefficient, the following regression is run separately for each instrument:

$$Abs\_Ret_{5s,t} = \alpha + \beta \cdot \text{Spread\_bps}_t + \varepsilon_t$$

Where:
- $\beta$ measures how much additional short-horizon move size is associated with a 1 basis point increase in spread.  
- The correlation $r$ between spread in basis points and 5-second absolute return provides a simple strength metric.  
- The **Volatility Multiplier (Q4/Q1)** compares average move size in wide-spread states against tight-spread states.

Together, these metrics quantify whether wider spreads coincide with meaningfully larger subsequent price movements.

---

### 4.4 Execution and Outputs

The analysis is implemented on the cleaned event stream from Section 2 using the Statistical L1 Proxy constructed above. The code will:

- Build an `l1_book` dataset containing proxy best bid, best ask, mid-price, spread, and 5-second forward returns for each instrument and second.  
- Compute a per-instrument summary table `spread_impact_df`, including the OLS $\beta$ coefficient, correlation $r$, and the Volatility Multiplier (Q4/Q1).  
- Produce a bar-chart visual showing average absolute 5-second move across the four spread quartiles.  
- Generate a Minute 39 stress comparison table, comparing each instrument’s average spread in Minute 39 against its session-average spread.

---

### 4.5 Event Drill-Down and Section Summary

Section 3 identified Minute 39 as the session peak in cancel-to-trade ratio and flagged it as a liquidity stress event.  
To connect Section 4 back to that earlier anomaly, the Minute 39 drill-down tests whether the message surge was accompanied by meaningful spread widening:

- For each instrument, compare average spread during Minute 39 to its own average spread across the full session.  
- Express the difference as a percentage change to highlight where liquidity deteriorated most.

Taken together, the spread regimes, regression outputs, and Minute 39 stress test address the four guiding questions from 4.1: whether wide-spread regimes coincide with larger short-horizon moves, how strong that effect is by instrument, whether it is directional, and whether the previously identified stress minute corresponds to a genuine deterioration in top-of-book liquidity.