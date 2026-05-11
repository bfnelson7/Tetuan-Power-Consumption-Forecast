# ⚡ Tetouan City Power Consumption Forecasting

> **A time series forecasting project using SARIMA and SARIMAX models to predict hourly electricity demand across three distribution zones in Tetouan City, Morocco.**
 
---

## Executive Summary
This project builds an automated 14 -day electricity demand forecast for Tetouan City's three power distribution zones, using a full year of hourly consumption readings across the city. The goal is simple: know how much power each part of the city will need, before it needs it.

The forecasting model accurately predicts the city's daily demand rhythm — including the sharp evening surge and the summer peak season — giving grid operators a reliable, data-backed view of what lies ahead rather than relying on rules of thumb or manual estimates.

The best-performing model correctly predicted demand to within roughly 1,040 kW of actual consumption on a typical hour — an error margin of around 3% relative to average demand — across a 14 -day forward window. Critically, adding weather data (temperature, humidity, wind speed) did not improve accuracy, meaning the model is both simpler to operate and more reliable in practice.

|Model| Hourly Error| Accuracy|
|-----|-------------|---------|
|SARIMA (recommended)| ~1,024kW| ~97%|
|SARIMAX (with weather)|~1,064kW|~96.7%|

At this accuracy level, the model is ready to support day-ahead procurement decisions, summer capacity planning, and evening peak demand management — reducing both the risk of supply shortfalls and the cost of holding unnecessary reserve capacity.

---
## Business Problem

Power distribution companies face two critical operational challenges that poor load forecasting directly aggravates:

1. **Under-provisioning** — insufficient generation or import capacity leads to unplanned outages, damaged industrial equipment, and regulatory penalties.
2. **Over-provisioning** — excess reserve capacity means wasted procurement spend and stranded asset costs.

Tetouan City's electricity network spans three geographically distinct distribution zones with different demand profiles. Without zone-level hourly forecasts, operators must rely on coarse estimates and conservative safety margins. This project addresses that gap by building a data-driven, statistically validated forecast model that can integrate into day-ahead dispatch and demand planning workflows.

**Dataset:** Tetouan City Power Consumption (UCI ML Repository)
- **52,416 rows** at 10-minute resolution
- **3 target zones:** Zone 1, Zone 2, Zone 3
- **Exogenous variables:** Temperature, Humidity, Wind Speed, General Diffuse Flows, Diffuse Flows
- **Period:** 1 January 2017 – 31 December 2017

---

## Key Findings & Recommendations

###  Key Findings

1. **Summer is the critical capacity period.** Zone 1 is the largest consumer and exhibits a clear **summer demand spike between July and September**, with peak values exceeding 50,000 kW — roughly 25% above the annual average. This aligns with cooling-driven demand escalation and has direct implications for summer capacity procurement planning. This is the primary window for grid stress events and requires dedicated capacity pre-positioning
<img width="744" height="285" alt="Screenshot 2026-05-11 at 6 37 47 PM" src="https://github.com/user-attachments/assets/34699f6d-01e6-4279-b32e-f82f17cdd66a" />


2. **Daily Load profile.** The daily load profile reveals a pronounced **bimodal pattern** common in urban distribution networks:
- **Off-peak trough:** 05:00–08:00 hrs (minimum demand, ~23,000 kW for Zone 1)
- **Morning ramp:** Consumption rises sharply from 08:00, driven by commercial and industrial activity
- **Evening peak:** Demand peaks between **19:00–21:00 hrs**, reaching ~43,000 kW for Zone 1 as residential and commercial overlap. Demand in this window is consistently the highest and most variable (widest interquartile range in the boxplot), making it the primary period for demand-side management interventions.
- **Zone hierarchy is consistent:** Zone 1 > Zone 2 > Zone 3 at all hours, with Zone 1 drawing roughly double Zone 3
<img width="744" height="276" alt="Screenshot 2026-05-11 at 6 38 35 PM" src="https://github.com/user-attachments/assets/42cb9b80-fd38-490d-b73b-0a11e14d0b6c" />
<img width="744" height="277" alt="Screenshot 2026-05-11 at 6 38 16 PM" src="https://github.com/user-attachments/assets/2587da6c-68c7-4c2d-9dfb-da23d66132bf" />


3. **Daily load cycle is the dominant forecast driver.** The 24-hour seasonality is highly regular and repeatable. SARIMA, using only this pattern, outperforms the weather-augmented SARIMAX, suggesting that at zone-aggregated level, weather effects are already embedded in historical consumption.

4. **Zone 1 carries disproportionate load.** At peak it draws nearly double Zone 3, and it is highly correlated with Zone 2. Any Zone 1 supply disruption has cascading inter-zone effects.

5. **The SARIMA model is production-ready.** With MAE of ~1,042 kW (~3% of mean demand) and clean residual diagnostics, the model meets the accuracy threshold for operational day-ahead scheduling.
   
### Recommendations

| Priority | Recommendation | Operational Impact |
|----------|---------------|-------------------|
| 🔴 High | Deploy SARIMA forecasts into the day-ahead dispatch system for Zone 1 | Reduces reserve margin requirement; improves cost efficiency |
| 🔴 High | Establish demand alert thresholds for 19:00–21:00 peak window | Enables pre-emptive load shedding or demand response activation |
| 🟡 Medium | Implement pre-summer capacity review (June trigger) using model's seasonal trend | Ensures infrastructure readiness for July–September surge |
| 🟡 Medium | Extend the model to Zones 2 and 3 using the same SARIMA structure | Provides system-wide coverage for full network dispatch planning |
| 🟢 Low | Investigate weather-driven demand events separately (extreme heat days) | Captures demand spikes that time-series models underestimate |

---
## Next Steps / Further Work

### Modelling Improvements

- **Extend to Zones 2 & 3:** Apply the same SARIMA framework to build parallel forecasting models for all three zones, enabling a consolidated city-wide load forecast.
- **Incorporate a second seasonality layer:** A weekly (168-hour) seasonal term would capture weekday vs. weekend demand differences, potentially reducing forecast error by 10–15%.
- **Explore TBATS / Prophet:** These models handle multiple seasonality natively and may outperform SARIMA on longer forecast horizons (30-45 days).
- **Machine learning benchmarking:** Test XGBoost and LSTM neural networks with engineered time features (hour, day-of-week, month, holiday flags) as alternative approaches for comparison.
- **Extreme event modelling:** Fit a separate model for anomalous high-demand days (e.g., heatwaves, public holidays) to supplement the base forecast.

### Data Enhancements

- **Extend dataset to multiple years** to capture inter-annual variation and improve seasonal parameter estimation.
- **Add holiday and event calendars** as binary exogenous features — public holidays in Morocco show distinctly different demand profiles.
- **Introduce real-time weather forecasts** (NWP model outputs) as forward-looking exogenous inputs to SARIMAX, replacing the in-sample weather observations used here.

### Operationalisation

- **Automate daily retraining** on a rolling window to keep the model fresh as consumption patterns evolve.
- **Build a forecast monitoring dashboard** tracking MAE and RMSE in real time against incoming actuals.
- **Define confidence interval thresholds** for procurement triggers: when the upper 95% CI exceeds a defined capacity limit, an automatic procurement signal is raised.

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| Python | Core language |
| Pandas | Data processing |
| Matplotlib / Seaborn | Visualisation |
| Statsmodels | SARIMA / SARIMAX modelling, ADF test, seasonal decomposition |
| SciPy | Box-Cox transformation |
| Scikit-learn | MAE / RMSE evaluation metrics |

---

## 📊 Data Source

UCI Machine Learning Repository — [Tetouan City Power Consumption Dataset](https://archive.ics.uci.edu/ml/datasets/Power+consumption+of+Tetouan+city)
