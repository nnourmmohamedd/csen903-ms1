# CSEN903 Milestone 1 – Formula 1 Driver Podium Prediction

This repository contains the corrected CSEN903 Milestone 1 notebook source and its supporting evidence. It predicts whether a driver will achieve a podium (top three) in a Formula 1 race using one row per driver-race.

## Kaggle validation

The existing public Kaggle notebook was updated in place to **Version 2** and Kaggle reports the complete execution as successful. The run used the Formula 1 dataset, CPU, and internet disabled. Its final integrated validation passed 33/33 checks. The public Version 2 source was pulled through Kaggle CLI and matches this repository notebook cell-for-cell across all 108 cells; the source SHA-256 is `8eaf48102c291c037e14fcd3913c2bb0624dd4447a9808d2404c5a0893be28e1`.

- [Public Kaggle Version 2](https://www.kaggle.com/code/nourmohameddd18/csen903-ms1)
A separate private checklist-validation run preceded publication; the linked evidence summaries are checked against the completed public Version 2 outputs.
- Dataset: [Formula 1 World Championship dataset](https://www.kaggle.com/datasets/rohanrao/formula-1-world-championship-1950-2020)
- Validation evidence: [`validation_evidence/`](validation_evidence/); the included summaries were checked against the completed public Version 2 outputs.

## Prediction point and data

The prediction unit is one unique `(raceId, driverId)`. The score is intended for after qualifying and final grid assignment but before race start. The feature set contains only pre-race inputs, including current-race grid and qualifying information plus prior driver, constructor, and championship history. Historical features are shifted to exclude the current race; post-race outcomes are not predictors.

The source data is the relational Formula 1 dataset linked above; the validated snapshot contains race years 1950–2024. In Kaggle, attach that dataset as an input. The notebook expects the Kaggle input directory for the dataset and writes generated evidence under `/kaggle/working/`.

## Workflow and assignment coverage

1. **Relational cleaning:** inspect raw tables, handle sentinels and semantic types, audit keys, preserve raw inputs, and document duplicate driver-race rows.
2. **Data engineering questions:** compare front-row starts with podiums by circuit/era; estimate home-versus-away outcomes adjusted by grid; compare mechanical retirements by constructor/era using starts as denominators.
3. **Feature engineering and split:** build pre-race features, use past-only driver/constructor histories, inspect coverage, and fit preprocessing on the chronological training partition.
4. **Predictive modeling:** compare Logistic Regression, Random Forest, and a shallow neural network with ROC-AUC, average precision (PR-AUC), and F1; select using validation data and evaluate the frozen models on test.
5. **Experiments:** six feature-group ablations and two preprocessing-order comparisons; test data is excluded from these experiments.
6. **Explainability:** validation-only permutation importance, SHAP, and a local LIME example. Attributions are not causal claims.
7. **Inference:** expose a function for complete or partially missing pre-race records and validate its outputs.

## Final corrected results

The chronological split contains 23,689 training rows (1950–2017), 1,620 validation rows (2018–2021), and 1,359 test rows (2022–2024). The schema has 31 raw predictors and 306 transformed features.

| Model | Validation ROC-AUC | Validation PR-AUC | Validation F1 | Threshold | Test ROC-AUC | Test PR-AUC | Test F1 |
|---|---:|---:|---:|---:|---:|---:|---:|
| Logistic Regression (`LR_C0.1_unweighted`) | 0.9374 | 0.7236 | 0.7193 | 0.36 | 0.9227 | 0.7011 | 0.6753 |
| Random Forest (`RF_4`) | 0.9409 | 0.7322 | 0.7273 | 0.75 | 0.9354 | 0.7121 | 0.6580 |
| Shallow Neural Network (`NN_3`) | 0.9337 | **0.7420** | 0.7138 | **0.81** | 0.9166 | 0.6710 | 0.6667 |

The shallow neural network was selected as the primary model by validation PR-AUC, before test evaluation. Its threshold of 0.81 was selected on validation F1. The Random Forest has higher test PR-AUC; test results did not change the validation-based model choice.

Stage 3 now displays a dynamic train-data before/after preprocessing example. For `q3_gap_to_best`, the training set has 21,511 missing raw values; median imputation uses 0.852 and leaves zero missing values. The observed raw range is 0?31.189; after the train-fitted imputer and scaler, the shown training range is about -2.527?88.260. The full per-feature table is in `validation_evidence/stage3_preprocessing_effect_summary.csv`. Stage 5 retrieves the selected Logistic Regression threshold (0.36) from Stage 4 for both preprocessing-order comparisons; their ranking scores are unchanged by the threshold, while threshold-based scores use the corrected value.

## Approved historical corrections

- **Driver-race podium target (N1):** retain the deterministic pre-race predictor row, but define the driver-race label as podium if any source result row for the driver-race finished in the top three. This changes 17 of 21 conflicting duplicate-group labels and raises the driver-race podium total from 3,378 to 3,395. Constructor form remains attributed to the constructor's own source entries.
- **Mechanical retirement classification (N2):** include verified `Mechanical` (7), `Drivetrain` (1), and two `Wheel nut` component failures; exclude one wheel-nut event supported as a pit-crew fitting error. Corrected count: 396/4,612 starts (8.59%), with non-starts excluded.

## Reproducing the Kaggle run

1. Create or open a Kaggle Notebook and add the Formula 1 dataset linked above as an input.
2. Use the Kaggle-managed Python environment with CPU enabled, GPU disabled, and internet disabled during execution.
3. For a reproducibility run, create a new private Kaggle notebook from `CSEN903_MS1_corrected.ipynb`, attach the dataset, use CPU, and disable internet.
4. Select **Run All** from a fresh session and confirm each stage summary and the final integrated validation passes. The compact summaries in `validation_evidence/` were checked against the completed public Version 2 outputs; the clean source notebook does not embed run outputs.

`requirements.txt` lists the third-party packages imported by the notebook. It deliberately omits package version pins because the available run evidence does not include a complete package lock. Kaggle metadata identified the managed image as `gcr.io/kaggle-images/python@sha256:81b1e7c4b4f0f2a7e0a33943c95754b1b7af6b30d65221633a284b6b601cd84a`; the validated run used this image with CPU and internet disabled.

## Validation and known limitations

The private checklist-patch run and public Version 2 both reported Stage 1: 13/13, PK audit: 14/14, FK audit: 23/23, Stage 2: 24/24, Stage 3: 77/77, Stage 4: 60/60, Stage 5: 14/14, Stage 6: 11/11, Stage 7: 12/12, and final integrated checks: 33/33.

Results are historical and observational. Nationality is a proxy for home race; constructor status coding is limited to 2014–2024; historical data and status labels can differ by era; and the test interval covers only 2022–2024. Model explanations describe associations used by the fitted model, not causes. Metrics do not guarantee future performance.
