# Store Item Demand Forecasting (2018 Q1)

Daily sales forecast for 10 stores × 50 items (500 series), 2018-01-01 to 2018-03-31 (90 days, 45,000 values), trained on 2013–2017 data. Based on the Kaggle [Store Item Demand Forecasting Challenge](https://www.kaggle.com/competitions/demand-forecasting-kernels-only).

## Approach: TopDown_Ens

1. Aggregate all 500 series into one daily total.
2. Forecast the total with two methods and average them 50:50:
   - **YoY (Year-over-Year):** same day last year (364 days earlier) × growth rate over the last 91 days
   - **Mult (Multiplicative):** regression on log(total) = trend + month + day of week
3. Split the forecast back to each store-item using its share of the total, by day of week, over the last 365 days.

## Backtest results

Rolling-origin backtest, 12 anchors (2015-01 to 2017-10), horizon 3 months.

| Model | SMAPE (%) | RMSE | MAE |
|---|---|---|---|
| **TopDown_Ens** (selected) | **12.59** | 7.81 | 6.05 |
| LightGBM (recursive) | 12.63 | 7.85 | 6.06 |
| TopDown_YoY | 12.68 | 7.92 | 6.09 |
| TopDown_Mult | 12.76 | 7.97 | 6.18 |
| TopDown (MSTL) | 12.99 | 8.19 | 6.30 |
| YearlyNaive | 17.99 | 11.07 | 8.51 |
| AutoETS (per series) | 19.21 | 13.00 | 9.93 |
| SeasonalNaive | 22.25 | 14.57 | 11.16 |
| *Oracle (true total known)* | *12.16* | – | – |

SMAPE = Symmetric Mean Absolute Percentage Error. RMSE and MAE are in units per day per store-item. The Oracle row is the ceiling of the top-down approach, not a model.

## Caveats

- In the Q1 (Jan–Mar) backtests, every model over-forecast by about 5% (TopDown_Ens: +5.3%), so the 2018 forecast may also run high.
- The p10/p90 intervals are per store-item per day and cannot be summed into an interval for the total.

## Files

| File | Description |
|---|---|
| `forecast_2018_TopDown_Ens.csv` | Main forecast: store, item, date, sales (45,000 rows) |
| `forecast_2018_TopDown_Ens_PI.csv` | Same forecast with p10 and p90 intervals |
| `slides.pdf` | Results report |
| `demand_forecast_topdown.ipynb` | Main code: EDA, backtest, forecast, Oracle test, error analysis, prediction intervals |
| `LightGBM_fixed.ipynb` | LightGBM (recursive) baseline, used only for comparison with top-down |

## How to run

1. Put `train.csv` (columns: `date, store, item, sales`) in the same folder as the notebooks.
2. Run `demand_forecast_topdown.ipynb` with Restart & Run All. It produces `backtest_results.csv`, `backtest_preds.csv`, `forecast_2018_TopDown_Ens.csv`, and `forecast_2018_TopDown_Ens_PI.csv`.
3. (Optional) Run `LightGBM_fixed.ipynb`, about 3–5 minutes. Run step 2 first, since it reads `backtest_results.csv` and `forecast_2018_TopDown_Ens.csv`.

**Environment:** Python 3.13.9, pandas 2.3.3, numpy 2.3.5, statsforecast 2.1.1, lightgbm 4.6.0, matplotlib 3.10.6