# 🌊 WaveCast

Predicting significant wave height from meteorological and oceanographic buoy data.


## Data

- **Source:** [NOAA National Data Buoy Center (NDBC)](https://www.ndbc.noaa.gov/), Station 41001, Standard Meteorological Data
- **Period:** 2022–2025 (hourly readings; note below on coverage)
- **Variables used:** wind speed, wind gust, wind direction (circularly encoded), atmospheric pressure, hour, month
- **Target:** `WVHT` — significant wave height, a statistical measure of sea state approximating the average height of the highest one-third of waves during an observation period

### Known data limitations

- Raw files stop early for 2024 (Sept) and 2025 (Oct) — coverage for those years is partial, not a full calendar year.
- NDBC encodes missing sensor readings with sentinel values (e.g. `99.00`, `999`, `9999.0`) rather than true nulls — handled explicitly per-variable during cleaning.
- Wave height (`WVHT`) is measured on an hourly cadence by this buoy's sensor, despite the underlying files being logged every 10 minutes for other variables (wind, pressure). 

## Methodology

```
NOAA/NDBC raw data
      ↓
Cleaning (sentinel handling, column selection, datetime indexing)
      ↓
Time-series analysis (seasonal decomposition, autocorrelation)
      ↓      ↓
Regression modeling 
      ↓
Evaluation & residual analysis 
```


## Time-series findings (so far)

- **Seasonal decomposition** shows a clear annual cycle: higher wave heights in winter months, calmer conditions in summer, consistent with expected storm-season patterns for this region.
- **Autocorrelation (ACF)** shows strong, slowly-decaying correlation extending past 48 hours — consistent with wave height's physical persistence (sea states build and decay gradually rather than resetting hour to hour). This justifies including lagged WVHT features in the regression stage.
- No clear short-cycle (daily) pattern was observed in ACF at short lags, consistent with wave height being storm-driven rather than diurnally driven.

## Modeling
**Model:** Multiple Linear Regression
**Features:** wind speed, wind gust, atmospheric pressure, wind direction (circularly encoded as `wind_dir_x`/`wind_dir_y`)
**Target:** significant wave height (WVHT)
**Train/test split:** chronological (80/20), not random.

| Metric | Value |
|--------|-------|
| MAE    | 0.46 m |
| RMSE   | 0.51 m |
| R²     | 0.55 |

### Key finding: the model systematically underestimates extreme wave heights

Segmenting error by wave-height range reveals a sharp performance gap:

| Wave height range | MAE |
|--------------------|------|
| < 3 m              | 0.35 m |
| ≥ 5 m              | 2.69 m |

Residuals (actual − predicted) are tightly centered near zero for typical sea states, but grow into a clear upward-sloping bias as actual wave height increases — the model increasingly underpredicts as conditions get more extreme, rather than erring randomly in both directions.

This is consistent with the model's feature set: wind speed, gust, pressure, and instantaneous direction capture typical wind-driven wave generation well, but likely miss what drives the most extreme sea states — sustained storm conditions building over many hours, and swell arriving from distant, already-passed weather systems (a signal more closely tied to `MWD`, which was deliberately excluded from features as a wave-state variable rather than a meteorological driver — see exclusions above). A linear model is also structurally limited in representing threshold or compounding effects that likely matter most at the extremes.



## Repository structure

```
wavecast/
├── README.md
├── requirements.txt
├── data/
│   └── clean/
├── notebooks/
│   ├── 01_exploration.ipynb
│   ├── 02_time_series_analysis.ipynb
│   └── 03_regression_modeling.ipynb   
└── src/                                 
```
