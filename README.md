# 🌊 WaveCast
This is a mini project predicting significant wave height from meteorological and oceanographic buoy data.

## question

Can meteorological and oceanographic conditions predict significant wave height (WVHT) ?

## Data

- **Source:** [NOAA National Data Buoy Center (NDBC)](https://www.ndbc.noaa.gov/), Station 41001, Standard Meteorological Data
- **Period:** 2022–2025 (hourly readings; note below on coverage)
- **Variables used:** wind speed, wind gust, wind direction (circularly encoded), atmospheric pressure, hour, month
- **Target:** `WVHT` — significant wave height, a statistical measure of sea state approximating the average height of the highest one-third of waves during an observation period

### Known data limitations

- Raw files stop early for 2024 (Sept) and 2025 (Oct) — coverage for those years is partial, not a full calendar year.
- A gap of several months between late 2024 and mid-2025.
- NDBC encodes missing sensor readings with sentinel values (e.g. `99.00`, `999`, `9999.0`) rather than true nulls — handled explicitly per-variable during cleaning.
- Wave height (`WVHT`) is measured on an hourly cadence by this buoy's sensor, despite the underlying files being logged every 10 minutes for other variables (wind, pressure). This was identified during cleaning and shaped the resampling strategy.

## Methodology

```
NOAA/NDBC raw data
      ↓
Cleaning (sentinel handling, column selection, datetime indexing)
      ↓
Time-series analysis (seasonal decomposition, autocorrelation)
      ↓
Feature engineering (circular encoding, lag features, time features)
      ↓
Regression modeling (baseline → linear → multiple → lag-augmented → gradient boosting)
      ↓
Evaluation & residual analysis (with focus on extreme-event performance)
```

### Why certain variables were excluded from the feature set

- `DPD`, `APD` (wave periods) were excluded as predictors — they are other measurements of the wave state itself (computed from the same sensor burst as WVHT), not independent meteorological drivers, and would risk leaking wave-state information into the model.
- `MWD` (mean wave direction) was treated as EDA-only, not a model feature, for the same reason — it's a property of the resulting waves, not a cause of them.
- `TIDE`, `DEWP` were dropped for weak/indirect physical relevance to wave height at this timescale.

## Time-series findings (so far)

- **Seasonal decomposition** shows a clear annual cycle: higher wave heights in winter months, calmer conditions in summer, consistent with expected storm-season patterns for this region.
- **Autocorrelation (ACF)** shows strong, slowly-decaying correlation extending past 48 hours — consistent with wave height's physical persistence (sea states build and decay gradually rather than resetting hour to hour). This justifies including lagged WVHT features in the regression stage.
- No clear short-cycle (daily) pattern was observed in ACF at short lags, consistent with wave height being storm-driven rather than diurnally driven.

## Modeling 

## Repository structure



