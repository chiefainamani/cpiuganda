# Model Selection Results: Uganda CPI Forecasting

Generated from `outputs/model_selection/` (reproduce all reported benchmarks with `uv run python -m forecasting.model_selection --with-sarima`).
Use this as the source for **Chapter 3 (3.2, 3.5, 3.6, 3.7)** and **Chapter 4 (4.3.2, 4.3.4, 4.4)** of the report.

## 1. What was compared

Four machine-learning candidates and three benchmarks, all scored by the same procedure.

| Model | Type | Idea in one line |
|---|---|---|
| Lasso | ML | Regularised linear regression; shrinks weak features to zero |
| Random Forest | ML | Average of many decision trees trained on random subsets |
| Gradient Boosting | ML | Shallow trees added one at a time to fix earlier errors |
| SVR (RBF) | ML | Support vector regression with a smooth kernel |
| SARIMA | Benchmark | Seasonal ARIMA refitted at every forecast origin |
| Naive-drift | Benchmark | Last 12 months' average growth projected forward |
| OLS-trend | Benchmark | The proposal's linear trend, refit on all data at each origin |

## 2. Method (for Chapter 3)

- **Data:** headline CPI, July 2017 to August 2026, 110 monthly values (UBOS). The project initially used the June release available when work began; this run adds the subsequently released July and August observations.
- **Target:** cumulative % growth of CPI over the next 1, 2 and 3 months (one model per horizon, the *direct* strategy). The forecast is converted back to a CPI level. Predicting growth instead of the level matters because tree models cannot predict outside the range they were trained on.
- **Features (12):** monthly % change lags 1, 2, 3, 6, 12; rolling mean of monthly change over 3, 6, 12 months; rolling std (6); annual inflation rate; month encoded as sine and cosine. All use only information known at the forecast date.
- **Validation:** walk-forward, expanding window. The first model is trained on 48 months (Jul 2017 to Jun 2021) and forecasts the next 3; the window then grows one month at a time through the last origin with three observed targets. This gives **60 forecast origins x 3 horizons = 180 forecasts per model**, including the 2022 inflation shock (annual CPI inflation peaked at 10.7% in Oct 2022).
- **Tuning without leakage:** hyper-parameters are chosen by forward-only cross-validation (`TimeSeriesSplit`, 3 splits) on the training data of each origin only, re-tuned every 12 origins.
- **Metrics:** MAPE (primary, same as the proposal's 2% target), RMSE, MAE, bias; reported overall, by horizon, and by economic period.
- **Significance:** Diebold-Mariano test with Harvey-Leybourne-Newbold correction.
- **Selection rule (fixed before looking at results):** rank the four ML models on (1) overall MAPE, (2) overall RMSE, (3) worst-period MAPE. Lowest average rank wins; ties are broken by overall MAPE.

## 3. Results (for Chapter 4)

### 3.1 Overall accuracy (lower is better)

| Model | MAPE_% | RMSE | MAE | Bias |
|---|---|---|---|---|
| SVR | 0.540 | 1.023 | 0.681 | -0.203 |
| GradientBoosting | 0.552 | 1.027 | 0.702 | -0.215 |
| RandomForest | 0.575 | 1.065 | 0.729 | -0.322 |
| Lasso | 0.580 | 1.128 | 0.732 | -0.149 |
| SARIMA | 0.591 | 1.074 | 0.749 | -0.310 |
| Naive-drift | 0.596 | 1.076 | 0.758 | -0.049 |
| OLS-trend | 2.022 | 3.276 | 2.591 | -2.577 |

### 3.2 MAPE by forecast horizon (%)

| Model | h=1 | h=2 | h=3 |
|---|---|---|---|
| SVR | 0.290 | 0.563 | 0.765 |
| GradientBoosting | 0.296 | 0.576 | 0.785 |
| RandomForest | 0.327 | 0.569 | 0.828 |
| Lasso | 0.327 | 0.565 | 0.846 |
| SARIMA | 0.326 | 0.609 | 0.839 |
| Naive-drift | 0.343 | 0.605 | 0.839 |
| OLS-trend | 1.900 | 2.023 | 2.144 |

### 3.3 MAPE by economic period (%)

Periods are defined by the forecast origin: pre-shock (to Dec 2021), shock (Jan 2022 to Jun 2023), normalisation (Jul 2023 on).

| Model | Normalisation (Jul 2023 on) | Pre-shock (to Dec 2021) | Shock (Jan 2022 - Jun 2023) |
|---|---|---|---|
| SVR | 0.275 | 0.366 | 1.121 |
| GradientBoosting | 0.348 | 0.329 | 1.035 |
| RandomForest | 0.355 | 0.333 | 1.096 |
| Lasso | 0.305 | 0.347 | 1.203 |
| SARIMA | 0.357 | 0.405 | 1.118 |
| Naive-drift | 0.332 | 0.281 | 1.231 |
| OLS-trend | 1.381 | 0.266 | 3.952 |

### 3.4 Ranking of the four ML models

| Model | MAPE % | RMSE | Worst-period MAPE % | Rank MAPE | Rank RMSE | Rank worst-period | Avg rank | Overall |
|---|---|---|---|---|---|---|---|---|
| SVR | 0.540 | 1.023 | 1.121 | 1.000 | 1.000 | 3.000 | 1.667 | 1.000 |
| GradientBoosting | 0.552 | 1.027 | 1.035 | 2.000 | 2.000 | 1.000 | 1.667 | 2.000 |
| RandomForest | 0.575 | 1.065 | 1.096 | 3.000 | 3.000 | 2.000 | 2.667 | 3.000 |
| Lasso | 0.580 | 1.128 | 1.203 | 4.000 | 4.000 | 4.000 | 4.000 | 4.000 |

SVR and Gradient Boosting tie on average rank (1.667). The tie-break (lower overall MAPE) selects **SVR**.

### 3.5 Is the winner significantly better? (Diebold-Mariano p-values)

| SVR vs | p (h=1) | p (h=2) | p (h=3) |
|---|---|---|---|
| GradientBoosting | 0.9340 | 0.5849 | 0.9399 |
| RandomForest | 0.0792 | 0.7910 | 0.3189 |
| Lasso | 0.1429 | 0.6021 | 0.2249 |
| SARIMA | 0.1301 | 0.5380 | 0.6882 |
| Naive-drift | 0.0105 | 0.3762 | 0.8411 |
| OLS-trend | 0.0000 | 0.0034 | 0.0225 |

These are two-sided HLN-corrected tests on squared errors; p-values are nominal and unadjusted for multiple comparisons. At the unadjusted 5% level SVR differs from OLS-trend at all three horizons and from Naive-drift at one month only. It is not significantly different from SARIMA or any other ML model at any horizon. The honest reading: all four ML models are close, SVR is nominally first, and the clearest statistical difference is from the OLS trend.

### 3.6 Feature importance (share of total; SVR has no native importance)

| Feature | Lasso | RandomForest | GradientBoosting |
|---|---|---|---|
| lag1 | 0.000 | 0.534 | 0.479 |
| lag2 | 0.000 | 0.054 | 0.026 |
| lag3 | 0.000 | 0.092 | 0.099 |
| lag6 | 0.000 | 0.012 | 0.017 |
| lag12 | 0.000 | 0.108 | 0.129 |
| roll_mean3 | 0.000 | 0.082 | 0.087 |
| roll_mean6 | 0.000 | 0.015 | 0.018 |
| roll_mean12 | 0.000 | 0.025 | 0.024 |
| roll_std6 | 0.000 | 0.036 | 0.043 |
| annual_pct | 0.000 | 0.019 | 0.034 |
| month_sin | 0.000 | 0.020 | 0.039 |
| month_cos | 0.000 | 0.005 | 0.006 |

Lasso shrank every feature to zero at the 1-month horizon in the final fit, i.e. it reduces to predicting the average growth. The tree models rely most on last month's change (lag1) and, to a lesser degree, lag 12 and lag 3.

### 3.7 Prediction intervals

Intervals are the 5th and 95th percentiles of the selected model's own walk-forward growth errors (percentage points of growth over the horizon): h=1 [-0.58, 0.73], h=2 [-1.24, 1.45], h=3 [-1.30, 2.62]. A hold-out check (quantiles learned on the first 60% of origins, tested on the last 40%) gave coverage of 100% at each horizon. Coverage above the 90% target suggests conservative (wide) intervals; the hold-out sample is limited, so this estimate is uncertain.

## 4. Final tuned parameters (SVR, fitted on all data)

{"1": {"C": 1.0, "epsilon": 0.1}, "2": {"C": 1.0, "epsilon": 0.1}, "3": {"C": 1.0, "epsilon": 0.1}}

## 5. Limitations to state in the report

1. **Small sample.** Only 110 months; the first model trains on about 35 usable rows. Rankings among the top models are within statistical noise.
2. **Naive-drift is a strong benchmark.** SVR beats it by a small margin (0.540% vs 0.596% MAPE) and not significantly at two or three months.
3. **Different protocol from the proposal's OLS figure.** The proposal's OLS MAPE of 1.52% used 82 steps starting at 24 months; here all models start at 48 months (60 origins) so the ML models have enough history. Under this shock-inclusive window OLS scores 2.022%. Never mix the two numbers in one table.
4. **Exploratory significance tests.** Diebold-Mariano p-values use the HLN correction but are unadjusted for the multiple model-by-horizon comparisons.
5. **Grid edges.** SVR chose C=1 (the smallest value tried) and epsilon=0.1 (the largest), so a wider grid may help.
6. **Horizon limit.** The ML forecast supports 1 to 3 months only; longer horizons would need new direct models.
7. **No exogenous drivers.** Features come from the CPI series itself; exchange rate, fuel prices or rainfall are natural future additions.
