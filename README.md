# Platelet Demand Forecasting and Inventory Simulation — Reproducibility Code (R2 v3)

Reproducibility notebooks for the manuscript **“Target-Based Platelet Demand Forecasting and Inventory Simulation During Rapidly Changing Demand: A Retrospective Single-Centre Study”** (manuscript ID: **ALM-2026-0435.R1**).

This repository provides a target-based forecasting and inventory-simulation workflow for daily platelet demand at a tertiary-care hospital. The analysis considers **eight product–ABO series**: platelet concentrate (PC) and apheresis platelet concentrate (APC), each stratified into A, B, O, and AB groups. The revised pipeline focuses on **consistent forecast-target dates, information availability, transparent model comparisons, and reproducible inventory analyses**.

> **Data availability:** The institutional Excel dataset is **not included** because it contains non-public hospital data. The notebooks cannot reproduce numerical results without authorized access to the original dataset. This code is intended for research and methodological reproducibility, **not direct clinical use**.

## Repository files

| File | Purpose |
|---|---|
| `Platelet_R2_Validated_Integrated_v3_FIXED(1).ipynb` | **Integrated analysis notebook** (49 cells, including 25 code cells). Contains the embedded analysis scripts, forecasting benchmarks, statistical comparisons, inventory simulations, figures, and validation checks. Suitable for inspecting individual methods and rerunning analyses interactively in Google Colab. |
| `Platelet_R2_Validated_Safe_Launcher_v3_FIXED_v2(1).ipynb` | **Self-contained safe launcher** (2 cells). Embeds a validated copy of the integrated notebook, verifies its SHA-256 hash, runs its executable cells in an isolated directory, and archives the executed notebook, results, and run metadata. It does **not** require the other `.ipynb` to be present in the Colab runtime. |

The `(1)` suffix is part of the uploaded filenames and may be removed when adding the notebooks to GitHub. Renaming the standalone files does not change their computational logic.

### Which notebook should I use?

- **For straightforward end-to-end reproduction:** use the **Safe Launcher**.
- **For methodology inspection, troubleshooting, or optional Chronos execution:** use the **Integrated Notebook**.
- Running both is **not required**. They provide alternative execution routes to the same embedded core analysis.

## Study design and evaluation protocol

| Item | Specification |
|---|---|
| Observation period | March 1, 2019–February 28, 2025 (**2,192 days**) |
| Training | March 1, 2019–February 29, 2024 (**1,827 days**) |
| Validation | March 1, 2024–August 31, 2024 (**184 days**) |
| Independent test | September 1, 2024–February 28, 2025 (**181 days**) |
| Forecast targets | PC-A, PC-B, PC-O, PC-AB, APC-A, APC-B, APC-O, APC-AB |
| Primary operational task | Nominal next-day (`h=1`) platelet-demand forecasting |
| Main performance measures | MAE, weighted MAE, MASE, target-specific rank; supplementary paired tests and confidence intervals |

### Temporal harmonization and fair SARIMA comparison

The revised workflow assigns training/validation/test membership according to the **date of demand being predicted (forecast-target date)**, rather than the forecast-origin date. For an outcome on date `d`:

- The primary LASSO, moving-average, and LASSO–moving-average blend predictors use last-confirmed demand through **`d−2`**.
- The main **SARIMA(1,1,1) × (1,0,1,7)** benchmark uses the **same `d−2` demand cutoff**, with parameters fitted on training data only and states updated sequentially without validation/test-period parameter refitting.
- The main SARIMA result is a **rolling, information-matched** comparison over the same 181 test target dates; it is **not** the earlier static 181-step forecast.
- The seasonal-naïve baseline refers to demand on target date **`d−7`**.

This is a nominal next-day ordering decision from origin `d−1`, but its most recent confirmed demand is from `d−2`; relative to that last confirmed observation, the SARIMA conditional prediction is effectively two steps ahead. **Matching evaluation dates alone is not sufficient to establish equal information availability across models.**

### Forecasting methods

The main benchmark covers **10 forecasting approaches**: historical mean, seasonal naïve, 7-day moving average (MA7), 14-day moving average (MA14), LASSO, Ridge, LightGBM, SARIMA, LASSO–MA7 blend, and LASSO–MA14 blend. The two blends use equal weights (`0.5/0.5`). The pipeline also contains exploratory sparse-demand/hurdle analyses.

The reproducibility revision documents the LightGBM benchmark settings (regression objective; 300 estimators; learning rate 0.03; 31 leaves; row/column subsampling 0.8; `subsample_freq=1`; `random_state=42`). Prophet and LSTM are **disabled** in the default core run.

## Requirements

- **Google Colab** with Python 3 and a Google account with access to the input Excel file.
- Google Drive access to the input path below for the **Safe Launcher**.
- The **Integrated Notebook** can alternatively request an interactive Excel upload if its standard Drive source is unavailable.
- The notebook installs or uses: `numpy`, `pandas`, `scikit-learn`, `scipy`, `lightgbm`, `statsmodels`, `matplotlib`, `openpyxl`, and `python-docx`. The Safe Launcher additionally uses `nbformat`, `nbclient`, and `ipykernel`.
- Optional Chronos execution requires PyTorch, `chronos-forecasting==2.3.2`, and access to the pretrained `amazon/chronos-t5-small` model.

The three previously audited core executions used **Python 3.13.18**, **LightGBM 4.6.0**, and **statsmodels 0.15.0**. Some dependencies are installed with compatible version ranges rather than exact pins; check the archived environment metadata when evaluating bit-level reproducibility.

## Input data

Expected filename:

```text
platelet_data_english_260529.xlsx
```

Expected Google Drive location:

```text
/content/drive/MyDrive/Platelets/platelet_data_english_260529.xlsx
```

The workbook contains daily platelet issuance, hospital operations, laboratory measurements, and related predictors. The pipeline expects the original variable names and date structure; arbitrary spreadsheets will **not** work without adaptation. Do **not** commit the institutional workbook, clinical extracts, individual-level records, or unrestricted run archives containing protected data.

## How to run

### Option A — Safe Launcher (recommended)

1. Put the authorized Excel file at `/content/drive/MyDrive/Platelets/platelet_data_english_260529.xlsx`.
2. Open **`Platelet_R2_Validated_Safe_Launcher_v3_FIXED_v2(1).ipynb`** in Google Colab.
3. Select **Runtime → Run all** and authorize Google Drive access.
4. The launcher checks the embedded integrated notebook's SHA-256 hash, creates a unique run directory, executes the integrated workflow, and archives the outputs.
5. Inspect `launcher_run_manifest.json` and confirm that `status` is `COMPLETED`.

The embedded integrated source notebook is validated against this SHA-256 checksum:

```text
df2dc7508600d551ba643e20c603c680bc67fc3fa142787e46f09f14299f24a8
```

The launcher deliberately modifies only the embedded notebook's **workspace setup, input-file setup, and archiving cells** for managed execution. The analytical cells remain the validated embedded version. It saves both the source notebook and the executed notebook for auditability.

### Option B — Integrated Notebook (interactive)

1. Open **`Platelet_R2_Validated_Integrated_v3_FIXED(1).ipynb`** in Google Colab.
2. Select **Runtime → Run all** and provide the authorized Excel workbook when prompted if it is not available in Google Drive.
3. The notebook creates its own unique `RUN_ID`, runs the embedded pipeline, performs model and inventory checks, and archives results if Drive saving is enabled.
4. To run the optional Chronos benchmark, find the cell containing `RUN_CHRONOS = False`, change the flag to `True`, and execute that cell with its prerequisite variables already loaded. **Chronos is not executed by default.**

The notebooks write helper Python modules (including `platelet_analysis_v2.py` and `platelet_S18_S19_benchmarks.py`) to the isolated **runtime**, so no separate `.py` download is required.

## Outputs

Completed Google Colab runs are archived under a unique run ID:

```text
/content/drive/MyDrive/Platelets/Reproducibility_Results/
└── R2_VALIDATED_V3_RUNS/
    └── <RUN_ID>/
        ├── analysis_r2_s18s19/
        ├── tables_r2_s18s19/
        ├── figures_r2_s18s19/
        ├── launcher_run_manifest.json               # Safe Launcher route
        ├── Platelet_R2_Validated_Integrated_v3_SOURCE.ipynb    # Safe Launcher route
        └── Platelet_R2_Validated_Integrated_v3_EXECUTED.ipynb  # Safe Launcher route
```

The **Integrated Notebook** direct route instead writes `R2_reproducibility_manifest.json` with its archive, and does not automatically create the launcher's source/executed notebook files.

Selected output files include:

| Output | Meaning |
|---|---|
| `analysis_r2_s18s19/02_all_predictions_long.csv` | Long-form daily forecasts used for the main model comparisons. |
| `analysis_r2_s18s19/Table2_S18_S19_corrected.csv` | Updated forecasting accuracy table, including matched SARIMA and corrected LightGBM. |
| `analysis_r2_s18s19/SARIMA_S19_rolling_h1_matched_predictions_181days.csv` | Daily information-matched SARIMA forecasts. |
| `analysis_r2_s18s19/SARIMA_S19_rolling_h1_matched_metrics_by_target.csv` | Matched SARIMA metrics for the eight target series. |
| `analysis_r2_s18s19/LightGBM_S18_predictions_181days.csv` | Independently reconstructed LightGBM daily forecasts. |
| `tables_r2_s18s19/Table3_With_Opening_Closing.csv` | Manuscript Table 3 simulated product-level ordering policies, with opening and closing inventories. |
| `analysis_r2_s18s19/Table3_Daily_Inventory_Trace.csv` | Day-level inventory trace and accounting audit for the Table 3 policies. |
| `analysis_r2_s18s19/Table3_Full_Inventory_Audit.csv` | Extended Table 3 inventory accounting summary. |
| `tables_r2_s18s19/S8_penalty_sensitivity_full_precision.csv` | Full-precision LASSO and Ridge penalty sensitivity. |
| `analysis_r2_s18s19/R2_integrity_check.json` | Test-date coverage and benchmark audit checks. |
| `figures_r2_s18s19/` | Generated forecasting and demand-pattern figures. |

Some Supplementary Tables (e.g. S4 and S5) and Table 4 are **printed as DataFrames in the executed notebook** rather than exported as standalone files with those exact table names. Review the saved executed notebook when checking those results.

### Inventory-simulation caveat

**Do not substitute** the core pipeline's exploratory `11_test_inventory_target_level.csv`, `12_test_inventory_product_level.csv`, or `13_test_inventory_daily_trace.csv` for the manuscript's Table 3/4 inventory outputs. The exploratory core simulator and the separately reconstructed manuscript simulator use **different inventory engines**.

- **Table 3:** seven forecasting policies with the same prespecified **2.5-day supply buffer** and **1.0 safety factor**, but policy-specific forecast-derived **opening inventories**. Its historical-mean ordering policy uses the pre-test **training + validation** period; the Table 2 historical-mean forecasting baseline is **training only**.
- **Table 4:** jointly selects the forecasting model and inventory buffer by validation-period shortage/wastage cost among **seven non-hurdle candidate forecasting methods** (historical mean, MA7, MA14, LASSO, Ridge, and the two blends). The manuscript-reconstruction code uses a days-of-supply grid of **2.0, 2.5, 3.0** and safety-factor grid of **0.9, 1.0, 1.1, 1.2**. SARIMA, LightGBM, Chronos, and hurdle models are **not** Table 4 policy candidates.
- The ABO substitution analysis in S6 is an **exploratory post-hoc allocation sensitivity**, not a fully coupled ABO inventory simulation. Supplementary Table S6 Part B is **not automatically regenerated as a complete standalone table** by these notebooks.
- Orders placed near the end of the test interval can remain **outstanding** at the boundary; inspect the inventory trace and audit rather than assuming every ordered unit has arrived.

## Chronos: optional supplementary benchmark (S7)

The integrated notebook includes an optional **zero-shot Chronos-T5-Small** evaluation (`amazon/chronos-t5-small`) over the same eight series and 181 test target dates. The optional cell uses `num_samples=20` and random seed `2026`, computes the predictive median, and prints target-wise MAE and MASE.

**Important distinctions:**

1. `RUN_CHRONOS = False` by default, including in the Safe Launcher's embedded integrated notebook. A default successful run **does not reproduce Supplementary Table S7**.
2. Chronos accesses observed demand through **`d−1`**, whereas the principal LASSO/MA/blend/matched-SARIMA comparison uses confirmed demand only through **`d−2`**. It is therefore **target-date aligned but not information-set matched**; differences must not be attributed solely to architecture.
3. The optional cell **prints** the S7 calculation; it does **not** automatically export a dedicated Chronos daily-prediction CSV. Save predictions and environment information separately if independent Chronos reproducibility is required.
4. Chronos may need model download access and additional computing resources. Its supplementary results were not part of the three validated default core reruns.

## Reproducibility and validation

The pipeline includes assertions or checks for:

- The fixed **181-day** independent test period across **eight** target series.
- Forecast-target-date boundaries and the corrected seasonal-naïve reference date.
- Training-only parameter estimation and consistent demand-information cutoffs in the matched SARIMA evaluation.
- SARIMA optimizer convergence, documented LightGBM parameters, and target-wise forecast coverage.
- Integer daily inventory mass balance, opening/closing stocks, and outstanding orders.
- Fixed primary LASSO/Ridge penalties, 5,000-replicate 7-day moving-block bootstrap, Diebold–Mariano paired comparisons, and separate penalty sensitivity analyses.
- Unique execution directories, source/input SHA-256 hashes, and non-overwriting archival manifests.

Three independent default **core-pipeline** runs were previously audited with identical source-code and input-data SHA-256 hashes. Their substantive predictions, tabulated analytical outcomes, policy selections, and figures agreed across runs. File-byte differences confined to execution durations or office-document metadata should not be interpreted as numerical disagreement. This audit **does not establish independent repeatability of Chronos**, which was disabled in those core runs.

### Known documentation caveat

One inherited Markdown description inside the integrated notebook refers to SARIMA as a “static 181-step” comparator. That wording is **outdated**: the **executed SARIMA benchmark and its saved run metadata** use `rolling_h1_availability_matched`. The earlier static result is retained only for historical comparison and must not be substituted into the manuscript's main Table 2.

## Data governance and limitations

- The study is **retrospective and single-centre**. Reproducing the analysis on the same institution's dataset does not constitute external validation.
- Inventory outcomes arise from a **simulation**, not observed prospective implementation. Different opening-stock assumptions can affect policy comparisons.
- Protected hospital data and any derived potentially sensitive files should remain in approved institutional storage; review files carefully before public release.
- Methodological research code is **not a validated clinical decision-support product**. Human oversight, local validation, operational testing, and institutional approval are required before any clinical deployment.
- The repository does not provide an unrestricted license unless a separate `LICENSE` file is added.

## Citation and contact

If you use this code, please cite the associated manuscript once a definitive journal citation is available:

> *Target-Based Platelet Demand Forecasting and Inventory Simulation During Rapidly Changing Demand: A Retrospective Single-Centre Study*. Annals of Laboratory Medicine, manuscript ALM-2026-0435.R1 (under review; bibliographic details to be updated upon publication).

For research questions or requests for access to restricted data, contact the corresponding author, subject to institutional policies and approval. **No protected institutional data are distributed in this repository.**
