# Predictive-Paradox027
# PGCB Power Demand Forecasting: Technical Summary

## 1. Project Overview
This project targets the development of a short-term load forecasting (STLF) model for the Bangladesh National Power Grid (PGCB). The goal is to predict hourly electricity demand (`demand_mw`) for the next hour based on historical grid data, meteorological conditions in Dhaka, and annual macroeconomic indicators.

### Performance Target
*   **Goal:** Mean Absolute Percentage Error (MAPE) < 3.0% on 2024 hold-out data.
*   **Achieved:** **2.68% MAPE** using an XGBoost Regressor.

---

## 2. Data Pipeline & Cleaning
The raw PGCB data contained significant anomalies that required a multi-stage cleaning routine:

| Issue | Resolution |
| :--- | :--- |
| **Manual Typos** | Values > 20,000 MW (impossible for the grid) were divided by 10 to correct factor-of-10 errors. |
| **Duplicates** | Multiple entries for the same timestamp were aggregated using the median. |
| **Temporal Gaps** | Strict hourly resampling was enforced. Gaps ≤ 3 hours were forward-filled. |
| **Renewables** | Solar and wind columns (NaN in historical data) were filled with 0 (pre-commissioning state). |
| **Data Merging** | Integrated hourly weather (Temp, Humidity, etc.) and annual Economic indicators (GDP, CPI). |

---

## 3. Feature Engineering
The model's success relies on capturing the rhythmic and environmental drivers of demand. All features were generated using a `shift(1)` logic to ensure **zero data leakage**.

### Core Feature Categories:
1.  **Autoregressive Lags:**
    *   `lag_1`: Demand at $t-1$ (captures immediate momentum).
    *   `lag_24`: Demand at $t-24$ (daily seasonality).
    *   `lag_168`: Demand at $t-168$ (weekly seasonality).
2.  **Temporal Encoding:**
    *   **Cyclical Sin/Cos:** Applied to `hour` and `month` to represent periodic continuity (e.g., 23:00 is close to 00:00).
    *   **Weekend Flag:** Binary indicator for Friday/Saturday (Bangladesh's weekend).
3.  **Environmental Interactions:**
    *   **Heat Stress Proxy:** `temp_c` * `humidity_pct`.
    *   **Cooling Degree Hours (CDH):** Max(0, `temp_c` - 24) to capture HVAC load surges.
4.  **Rolling Statistics:**
    *   24-hour mean and standard deviation of demand to capture recent volatility.

---

## 4. Modeling Strategy
I employed an **XGBoost Regressor**, chosen for its ability to handle high-dimensional tabular data and non-linear interactions (e.g., temperature spikes and peak-hour demand).

*   **Training Period:** 2015-04-19 to 2023-12-31.
*   **Test Period:** 2024-01-01 to 2024-12-31 (Entirely unseen during training).
*   **Validation:** Last 10% of training data used for early stopping to prevent overfitting.
*   **Hyperparameters:** `learning_rate=0.04`, `max_depth=6`, `subsample=0.8`, `colsample_bytree=0.8`.

---

## 5. Key Results (2024 Hold-out)
The model demonstrates high precision in tracking daily peaks and nocturnal valleys.

*   **MAPE:** 2.68%
*   **RMSE:** ~517 MW
*   **R² Score:** 0.955

### Feature Importance Summary
The model identifies the **1-hour lag** as the dominant predictor, followed by the **24-hour lag** and **Temperature**. This aligns with grid behavior: current load and yesterday's routine are the strongest baseline indicators, while weather provides the necessary correction for HVAC demand.

---

## 6. Conclusion
The pipeline successfully delivers a human-readable and high-precision forecasting tool. The resulting MAPE of 2.68% is well below the 3% requirement, making it a viable model for operational grid management. 

Future iterations could be further improved by:
1.  Adding a **Public Holiday Dataset** (Eid typically causes major demand drops).
2.  Incorporating **Region-Specific Weather** (averaging stations across Bangladesh).
3.  Implementing a **Multi-Step Recursive Forecast** for longer horizons (e.g., 24-48 hours).
