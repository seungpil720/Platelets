# Platelet demand forecasting and inventory simulation (simplified reproduction)

This repository provides a simplified, publicly available reproduction of the core
forecasting and inventory-simulation pipeline described in the manuscript
"Target-Based Platelet Demand Forecasting and Inventory Simulation During Rapidly
Changing Demand." It is intended as an illustration of the methodology, not a
complete reproduction of every reported result (see "Scope" below).

## Scope

`platelet_forecast_inventory.py` implements:

- data loading and construction of the eight product-ABO forecasting targets
  (PC-A/B/O/AB, APC-A/B/O/AB)
- demand-pattern diagnostics (average demand interval, squared coefficient of
  variation among nonzero days, smooth / intermittent / erratic / lumpy
  classification)
- baseline forecasts: historical mean, seasonal naive, 7-day and 14-day moving
  average
- LASSO and Ridge regression forecasts, using the same fixed, prespecified
  penalty as the primary analysis (**LASSO alpha=0.05; Ridge alpha=5.0**) — no
  cross-validated penalty selection (`LassoCV`/`RidgeCV`) is used anywhere in
  this repository or in the primary analysis
- LASSO-moving-average adaptive blends (50/50 with MA7 and MA14)
- a discrete-unit, age-structured inventory simulation (integer on-hand units,
  ceiling-based order-up-to ordering rule), illustrated with a single example
  policy, with an exact daily integer mass-balance check built in

### Not included

This repository does **not** include: LightGBM, SARIMA, Prophet, LSTM, or
hurdle / two-stage models; the validation-based grid search over
(model x days_supply x safety_factor) used to select the target-level inventory
policies reported in Table 4; the moving-block bootstrap and Diebold–Mariano
comparisons (Supplementary Table S4); the rolling-origin analysis (Supplementary
Table S5); or the ABO-compatible substitution sensitivity analysis (Supplementary
Table S6). These are internal analyses that depend on the institution's full
clinical data pipeline and are not required to reproduce the core methodology.
Because the feature set here is a reduced illustrative version of the full
predictor set described in the Supplementary Methods, exact MAE values will
differ slightly from the published Table 2, though the relative ranking of
models is preserved.

## Data availability

The underlying hospital-level source data are not publicly available because
they contain institution-level clinical and operational information and are
subject to institutional data-sharing and privacy requirements. This repository
contains analysis code only. Researchers seeking access to the underlying data
for legitimate research purposes may contact the corresponding author, subject
to institutional approval and applicable data-sharing requirements.

## Usage

```bash
pip install -r requirements.txt
python platelet_forecast_inventory.py
```

The script expects a data file named `platelet_data_english_260529.xlsx` (not
included in this repository) in the working directory, in the same transposed
layout as the institutional source file (metadata columns followed by one
column per calendar day). Supply your own file in this layout to run the
pipeline end to end.
