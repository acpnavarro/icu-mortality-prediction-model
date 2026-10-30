# Outputs

This folder is created by the pipeline. Running the notebooks in order populates it with intermediate
and final artifacts (CSV tables, variable-set JSON, model checkpoints, packaged calculators).

**None of these files are committed** — they may contain derived patient-level data. The `.gitignore`
at the repository root blocks everything in this folder.

Expected layout after a full run:

```
outputs/
├── phase1/           # integrated cohort, audit tables, prepared Model A base
├── phase2/           # engineered features, Table 1, variable-set lists
├── phase3/           # baseline performance tables
├── phase4/           # boosting comparison, DeLong, calibration, SHAP, checkpoints
├── phase5/           # fairness, classification metrics, recalibration, robustness
├── phase5_adjusted/  # Part II artifacts (adjusted outcome definition)
└── phase6/           # calculators (.joblib), TRIPOD-AI/PROBAST tables
```
