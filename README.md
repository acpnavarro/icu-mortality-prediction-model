# ICU Mortality Prediction Model 

A machine learning pipeline that predicts **hospital mortality in ICU patients** from data available in
the **first 24 hours** of admission, developed on a single-center database of **~7,500 ICU admissions**,
and benchmarked against the **SAPS3** standard mortality equation.

This repository contains the **complete analysis pipeline** — from raw spreadsheet integration to a
packaged risk calculator and TRIPOD-AI/PROBAST reporting — as a series of numbered, self-documenting
Jupyter notebooks. **No patient data is distributed here** (see [Data availability](#data-availability)).

---

## Study overview

| | |
|---|---|
| **Primary outcome** | Hospital mortality |
| **Secondary outcome** | ICU mortality |
| **Cohort** | Index admissions (one per patient), ~5,700 patients; sensitivity analysis with all ~7,500 admissions |
| **Temporal contract** | Only variables available up to 24 h after ICU admission may enter as predictors |
| **Benchmark** | SAPS3 standard-equation death probability |
| **Validation** | Temporal (train ≤2022, test ≥2023) as primary; random by patient as stability reference |
| **Models** | Regularized logistic regression (baseline) → XGBoost / LightGBM / CatBoost |
| **Interpretability** | Global and local SHAP; simplified 30-variable score |
| **Reporting** | TRIPOD-AI checklist; PROBAST risk-of-bias assessment |

**Headline result (temporal validation, test set ≥2023):** the best gradient boosting model on raw
clinical components (Set 2) reached **AUROC ≈ 0.91** against **SAPS3 ≈ 0.83**, with the advantage
confirmed by the DeLong test (p < 0.001), and superior net clinical benefit on decision-curve analysis.

## Results

Temporal validation (training: admissions before 2023, n = 4,276; test: admissions from 2023 onwards, n = 1,400; hospital mortality 15.6% and 12.2%, respectively).

| Model | Predictors | AUROC | AUPRC | Brier |
|---|---|---|---|---|
| SAPS3 (benchmark) | standard equation | 0.834 | 0.475 | 0.085 |
| Logistic regression | Set 1 (severity scores) | 0.827 | 0.468 | 0.085 |
| LightGBM | Set 1 (severity scores) | 0.845 | 0.476 | 0.083 |
| Logistic regression | Set 2 (raw clinical components) | 0.850 | 0.530 | 0.081 |
| **XGBoost** | **Set 2 (raw clinical components)** | **0.907** (95% CI 0.881–0.932) | **0.674** | **0.063** |
| 30-variable calculator | Set 2, top 30 by SHAP | 0.906 | n/a | 0.064 |

- DeLong test against SAPS3: Set 2 XGBoost ΔAUROC = +0.073, p < 0.001; Set 1 LightGBM ΔAUROC = +0.011, p = 0.24 (no significant gain when the model only sees the composite scores).
- Calibration of the Set 2 model on the test set: intercept 0.27, slope 0.97.
- The 30-variable calculator does not differ significantly from the full 132-variable model (DeLong p = 0.67).
- Under the adjusted outcome definition (patients transferred to another hospital excluded, test n = 1,277), the full model keeps AUROC 0.907 and the advantage over SAPS3 remains significant (p < 0.001).

The 95% CI for the XGBoost AUROC is derived from the DeLong variance computed in Phase 6.

## Pipeline

```
Phase 1  Cohort construction          Sheet integration, temporal contract, patient identity
                                      (connected components), index-admission cohort
Phase 1b  All-admissions base         Same logic generalized to every admission (sensitivity input)
Phase 2  Feature engineering          Comorbidity groups, physiological ratios, secondary diagnoses
                                      (Folha8, positional alignment), missingness indicators, Table 1
Phase 3  Baseline model               Regularized logistic regression on two variable sets,
                                      temporal + random validation, vs. SAPS3
Phase 4  Gradient boosting            XGBoost/LightGBM/CatBoost, hyperparameter tuning
                                      (StratifiedGroupKFold), SHAP, DeLong, calibration, decision curve
Phase 5  Full validation              Fairness by subgroup, classification metrics, post-hoc
                                      recalibration, local SHAP, robustness with repeated admissions,
                                      transfer-bias quantification (adjusted outcome definition)
Phase 6  Calculator & reporting       Restricted 30-variable risk calculator, TRIPOD-AI checklist,
                                      PROBAST assessment
```

### Key methodological safeguards

- **Temporal contract (24-h cutoff).** Anything describing the rest of the hospital stay is treated as an
  outcome or leakage and excluded from predictors.
- **Patient identity via connected components.** Admissions are linked by `EpimedCode` **or**
  `MedicalRecord` (directly or through chains) — the most conservative definition against train–test
  leakage. All cross-validation is grouped by patient.
- **Positional recovery of ambiguous pairs.** 555 admissions with a duplicated key were recovered by
  aligning on original row position, with key verification at every step; failures are documented, never
  silenced.
- **SAPS3 reserved as a yardstick.** The SAPS3 probability never enters as a predictor, keeping the
  comparison fair.
- **Transfer-bias quantification.** Patients transferred to another hospital are coded as survivors by
  the primary outcome definition; Phase 5 (Part II) rebuilds the cohort under an adjusted definition
  (transferred excluded) and quantifies the impact with the DeLong test.
- **No imbalance correction.** Documented decision, consistent across logistic regression and boosting.
- **Recalibration decided by bootstrap.** Platt scaling is fitted on out-of-fold training predictions
  only, and only if the 95% bootstrap CI of the calibration intercept/slope indicates systematic deviation.

## Repository structure

```
├── notebooks/            # 01–07: the pipeline, in execution order
├── data/                 # empty: place your source spreadsheet here (never committed)
├── outputs/              # created by the pipeline when run
├── .github/workflows/    # privacy check: fails if a notebook carries outputs or a data file is committed
├── requirements.txt
├── LICENSE               # MIT
└── README.md
```

| Notebook | Phase | Reads from | Writes to |
|---|---|---|---|
| `01_cohort_construction` | 1 | `data/` | `outputs/phase1` |
| `02_sensitivity_all_admissions` | 1b | `data/` | `outputs/phase1b` |
| `03_feature_engineering` | 2 | `outputs/phase1` | `outputs/phase2` |
| `04_baseline_model_development` | 3 | `outputs/phase2` | `outputs/phase3` |
| `05_model_comparison_calibration` | 4 | `outputs/phase2`, `outputs/phase3` | `outputs/phase4` |
| `06_unified_validation_robustness` | 5 | `outputs/phase1b`, `outputs/phase2`, `outputs/phase4` | `outputs/phase5`, `outputs/phase5_adjusted` |
| `07_calculator_reporting_tripod` | 6 | `outputs/phase2`, `outputs/phase4`, `outputs/phase5_adjusted` | `outputs/phase6` |

## How to run

```bash
git clone https://github.com/acpnavarro/icu-mortality-prediction-model.git
cd icu-mortality-prediction-model

python -m venv .venv && source .venv/bin/activate   # or conda
pip install -r requirements.txt

jupyter lab   # then open notebooks/01_cohort_construction.ipynb and run in order
```

1. Place the source spreadsheet in `data/` and set `FILE_PATH` in the first notebook accordingly.
2. Run the notebooks in numerical order. Each phase reads the outputs of the previous one from `outputs/`.
3. Phase 1b must run after Phase 1 and before Phase 5 (Phase 5 reads its output for the sensitivity analysis).

The notebooks were developed in Google Colab; they run anywhere with the dependencies installed.

## Data availability

**No patient data is contained in this repository.** The source database is a proprietary hospital
electronic health record extract and cannot be shared — not even in derived form — due to privacy and
ethical restrictions. The notebooks are published to document the **methods and analysis pipeline**:
every transformation, validation decision, and statistical test is fully readable in the code and in the
markdown narrative.

`data/README.md` describes the expected input schema so the pipeline can be rerun on an authorized copy
of the data.

## Author

**Anna Carolina Pousas Navarro**

## License

This project is licensed under the [MIT License](LICENSE).
