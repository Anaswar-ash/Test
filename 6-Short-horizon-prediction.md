# Section 6: Short-Horizon Prediction — Implementation Plan

This Plan.md provides a definitive, professional roadmap for Section 6. It incorporates a **Segmented Modeling** strategy so I can demonstrate both quantitative rigor and market domain knowledge.

---

## 6.1 Goal & Strategy

**Objective:**  
Build a short-horizon classifier to predict the sign (direction) of the next 5-second mid-price move.

**Segmented Approach:**  
To account for the fundamental differences between equity index volatility and FX/Gold dynamics, I implement two separate models:

- **Model A (NAS100):** Equity index microstructure.
- **Model B (Macro Pool):** EURUSD, GBPUSD, USDJPY, XAUUSD.

**Logic – Signal Portability:**  
This setup explicitly tests **signal portability**: does a simple microstructure signal generalise across assets, or is its efficacy asset-specific?

**Primary Metric:**  
Performance is judged by **improvement over a Majority-Class Baseline**, ensuring that any reported accuracy reflects genuine predictive edge rather than simply learning class imbalance or trend persistence.

---

## 6.2 Feature Construction

All features are derived from the cleaned L1 proxy `l1_4_3` and are strictly backward-looking at time \(t\) to prevent look-ahead bias.

| Feature Category  | Column Name  | Description                                           |
|-------------------|-------------|-------------------------------------------------------|
| Liquidity Cost    | `spread_bps` | Instantaneous cost of crossing the spread.           |
| Momentum          | `past_ret_5s` | Lagged 5s mid-return (previous `ret_5s_bps`).        |
| Regime Context    | `vol_60s`    | Rolling 60s standard deviation of returns.           |
| Seasonality       | `minute`     | Integer minute to capture intra-session patterns.    |
| Target (\(y\))    | `target`     | Binary: 1 if `ret_5s_bps > 0`, else 0.               |

**Key Design Choices:**

- **Basis Points (bps):** Returns and spreads are expressed in bps so that a 10 bps move in NAS100 is directly comparable to a 10 bps move in XAUUSD.  
- **No Level 2 (Order Book):** Level 2 data is excluded due to data integrity limitations identified earlier; the focus is a robust L1 proxy rather than fragile depth metrics.

---

## 6.3 Methodology & Statistical Hygiene

**Model Choice:**  
- **Logistic Regression** with `class_weight='balanced'`.  
- Rationale: provides interpretable coefficients (e.g. how wider spreads or higher volatility affect the probability of an up-move) while handling class imbalance.

**Data Splitting:**  
- **70% Train / 30% Test**, strictly **chronological**.  
- **Constraint:** No random shuffling. This avoids look-ahead bias and respects temporal causality: the model is trained on the past and tested on future data.

**Evaluation Baseline:**  
- **Majority-Class Baseline:** always predicts the most frequent training label.  
- This is the benchmark to beat; all model performance is reported as accuracy relative to this baseline to highlight true edge.

---

## 6.4 Implementation Pipeline (Python Tasks)

1. **Preparation**
   - Start from `l1_4_3`.
   - Derive:
     - `past_ret_5s` = groupby(`sym`) on `ret_5s_bps`.shift(1).
     - `vol_60s` = rolling 60-second std of `ret_5s_bps` per `sym`.
     - `minute` = `ts_int // 60`.

2. **Segmentation**
   - **NAS100 segment:** `sym == 'NAS100'`.
   - **Macro Pool segment:** `sym in ['EURUSD', 'GBPUSD', 'USDJPY', 'XAUUSD']`.

3. **Engine Function**
   - Implement `run_segmented_model(df_segment, label)` that:
     - Cleans NAs for required features/target.
     - Builds feature matrix: `spread_bps`, `past_ret_5s`, `vol_60s`, `minute` (+ optional one-hot `spread_regime`).
     - Applies a 70/30 chronological split.
     - Computes **Majority-Class Baseline** accuracy on the test set.
     - Fits logistic regression with `class_weight='balanced'`.
     - Returns test accuracy for both baseline and model.

4. **Execution**
   - Call `run_segmented_model` for:
     - NAS100 segment (Model A).
     - Macro Pool segment (Model B).
   - Store results in a small DataFrame:
     - Rows: `NAS100 (Index)`, `Macro Pool (FX/Gold)`.
     - Columns: `Baseline Accuracy`, `Model Accuracy`.

5. **Visualisation**
   - Produce a grouped bar chart comparing:
     - NAS100: Model vs Baseline accuracy.
     - Macro Pool: Model vs Baseline accuracy.
   - Optionally add value labels on bars and a horizontal reference line at 0.5.

---

## 6.5 Discussion (100–200 Words)

The final write-up will cover three pillars:

### Assumptions

- The model assumes **local stationarity** of the microstructure regime within the one-hour session and within the train/test split window.  
- Logistic regression imposes a **linear relationship** between features and the log-odds of an up-move; non-linear effects are not fully captured at this stage.

### Feature Choices

- **Basis Points as a universal scale:** Using bps for returns and spreads makes signals comparable across NAS100, FX, and Gold, which is essential for cross-asset “portability” tests.  
- **L1-only microstructure:** Level 2 order book depth is deliberately excluded due to earlier data integrity issues; the focus is a robust, interpretable L1 proxy rather than potentially noisy depth fields.

### Scalability

- **Data:** Feature engineering would be migrated to a distributed framework such as **Dask** or **PySpark** to handle terabytes of tick data.  
- **Architecture:** The baseline logistic regression would be upgraded to **Gradient Boosting** (XGBoost/LightGBM) to capture non-linear interactions while preserving per-symbol or per-segment models.  
- **Deployment:** In production, this setup would evolve toward **online or periodic retraining**, allowing the models to adapt to drifting market regimes and changing microstructure.

---

## Why this "Cracks" the Project

- **Intellectual Honesty:** I report **edge over a baseline**, not raw accuracy in isolation.  
- **Market Awareness:** I acknowledge that different assets have different microstructure and treat NAS100 separately from FX/Gold.  
- **Statistical Defense:** By explicitly addressing class imbalance, look-ahead bias, and chronological splitting, I pass the core quantitative sanity checks interviewers care about.

Next step: Implement the `run_segmented_model` function on `l1_4_3`, run both segments, and then plug the actual accuracy numbers into Section 6 results and discussion.