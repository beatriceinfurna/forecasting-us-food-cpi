# 📈 Business Forecasting — U.S. Lodging CPI

**Course:** Business Forecasting | IESEG School of Management
**Professor:** Filip Van den Bossche
**Student:** Beatrice Maria Infurna
**Date:** April 2026
**Tools:** Python · statsforecast · statsmodels · scikit-learn · pandas · matplotlib

---

## 📌 Project Overview

This project applies time series forecasting methods to the **Consumer Price Index (CPI) for Lodging Away from Home** in the U.S. (series `CUUR0000SEHB`), a monthly index published by the **U.S. Bureau of Labor Statistics (BLS)** and retrieved via FRED.

The series spans from December 1997 to December 2024 (~26 years) and exhibits clear monthly seasonality, a long-term upward trend, and several structural shocks — the 2001 recession, the 2008 financial crisis, and the dramatic COVID-19 collapse in 2020 followed by an inflation-driven surge in 2021–2023.

---

## 📂 Dataset

| Field | Detail |
|-------|--------|
| **Series** | `CUUR0000SEHB` — CPI for Lodging Away from Home |
| **Source** | [U.S. Bureau of Labor Statistics via FRED](https://fred.stlouisfed.org/series/CUUR0000SEHB) |
| **Frequency** | Monthly |
| **Period used** | December 1997 – December 2024 (325 observations) |
| **Train / Test split** | Jan 1998 – Dec 2021 (train) / Jan 2022 – Dec 2024 (test, ~3 years) |

---

## 🔍 Analysis Steps

1. **Train / Test Split** : ~87% train / ~13% test (36 months, covering 3 full seasonal cycles)
2. **Exploratory Data Analysis** : Time plot, seasonal plot, seasonal subseries plot, ACF/PACF
3. **Transformations** : Box-Cox assessment (optimal λ ≈ 0.375 → log transformation applied); intervention dummies for COVID-19 (Apr 2020–Apr 2021) and the 2008 Financial Crisis (Oct 2008–Dec 2009)
4. **ETS Models** : Three candidates compared: AutoETS, ETS(M,A,M), ETS(M,Ad,M)
5. **ARIMA Models** : ACF/PACF-guided selection, AutoARIMA, and a dynamic regression model with intervention dummies
6. **Model Evaluation** : Test set accuracy (RMSE, MAE, MAPE, MASE), residual diagnostics, Ljung-Box test
7. **Out-of-Sample Forecasts** : 24-month forecast for 2025–2026 using the final model
8. **Extra: Machine Learning** : Gradient Boosting (GBM) and Neural Network Auto-Regressor (NNAR) comparison

---

## 📊 Key Results

### Model Comparison — Test Set (Jan 2022 – Dec 2024)

| Model | Test RMSE | Test MAPE | Ljung-Box p-value | White Noise Residuals |
|-------|-----------|-----------|-------------------|-----------------------|
| **ETS(M,A,M)** | **4.11** | **1.71%** | 0.29 | ✅ Yes |
| AutoARIMA | 5.62 | 2.33% | 7.4e-07 | ❌ No |
| Gradient Boosting | 19.37 | 8.75% | — | — |
| NNAR | 60.21 | — | — | — |

> AIC values between ETS and ARIMA are not directly comparable : ETS was fitted on the original scale, ARIMA on log-transformed data.

### Final Model: ETS(M,A,M)
Measurement:  y_t = (l_{t-1} + b_{t-1}) * s_{t-m} * (1 + e_t)

Level:        l_t = (l_{t-1} + b_{t-1}) * (1 + alpha * e_t)

Trend:        b_t = b_{t-1} + beta * (l_{t-1} + b_{t-1}) * e_t

Seasonal:     s_t = s_{t-m} * (1 + gamma * e_t)

Selected because it achieved the lowest test error (RMSE = 4.11, MAPE = 1.71%), passed the Ljung-Box test (p = 0.29 ✅), and its multiplicative structure naturally handles the increasing seasonal amplitude over time , no manual transformation or dummy variables needed.

### Best ARIMA Model: ARIMA(0,1,1)(0,1,1)[12] + Intervention Dummies
(1-B)(1-B^12) log(y_t) = -0.0793COVID_t - 0.0014Crisis2008_t + (1 - 0.274B)(1 - 0.674B^12) e_t
| Parameter | Estimate | p-value | Interpretation |
|-----------|----------|---------|----------------|
| theta_1 | -0.274 | < 0.001 | MA(1) — significant |
| Theta_1 | -0.674 | < 0.001 | Seasonal MA(1) — significant |
| beta_COVID | -0.079 | < 0.001 | ~7.6% price drop during COVID |
| beta_crisis2008 | -0.001 | 0.90 | Not significant |

### Out-of-Sample Forecasts (2025–2026)

| Period | Forecast |
|--------|----------|
| Summer peaks (Jun–Aug) | CPI ~195–205 |
| Winter troughs (Jan–Feb) | CPI ~175–185 |
| Annual growth | ~2–3% per year |

---

## 📁 Files

| File | Description |
|------|-------------|
| `forecasting_us_lodging_cpi.ipynb` | Full analysis notebook |
| `CUUR0000SEHB.csv` | Raw dataset from FRED |

---

## 🚀 How to Run

```bash
git clone https://github.com/beatriceinfurna/forecasting-us-lodging-cpi
cd forecasting-us-lodging-cpi
jupyter notebook forecasting_us_lodging_cpi.ipynb
```

```bash
pip install pandas numpy matplotlib seaborn scipy statsforecast statsmodels pmdarima scikit-learn neuralforecast
```

Place `CUUR0000SEHB.csv` in the same folder as the notebook before running.

---

## 💡 Key Takeaways

- **ETS outperformed ARIMA** : the multiplicative seasonal structure handled increasing variance without needing extra complexity
- **ML models underperformed significantly** : GBM can't extrapolate beyond training range; ~290 obs is too few for NNAR
- **Always check residual diagnostics** : the ARIMA model looked accurate on test metrics but failed the Ljung-Box test
- **COVID-19 reduced lodging prices by ~7.6%**; the 2008 financial crisis had no statistically significant effect
