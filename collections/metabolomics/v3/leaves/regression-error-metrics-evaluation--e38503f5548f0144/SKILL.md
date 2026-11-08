---
name: regression-error-metrics-evaluation
description: Use when trained regressors (e.g., RT prediction from molecular descriptors)
  must be validated on a held-out test set and compared via RMSE, R2, MAE, and an
  error-within-threshold fraction.
when_to_use_negative:
- The prediction problem is classification (e.g., metabolite detected/not-detected)
  rather than RT regression — use classification metrics instead
- Authentic analytical/stable-isotopic-labeled standards are available for the metabolites,
  so direct standard verification rather than in silico RT prediction is the appropriate
  validation channel
- There are no experimentally measured RTs for any compounds, so neither training
  nor held-out evaluation is possible
edam_operation: http://edamontology.org/operation_0366
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: XGBoost
  role: Best-performing regressor whose test-split RT predictions are scored with
    RMSE/R2/MAE and the 1-min error-fraction criterion
- name: scikit-learn Random Forest
  role: Comparison regressor trained on the same 80:20 split and scored with the same
    error metrics
- name: BRNN (Bayesian regularized neural network)
  role: Third comparison regressor evaluated alongside XGBoost and Random Forest
- name: RDKit
  role: Computes molecular descriptors from SMILES/InChIKey strings to form the regression
    feature matrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:caee56f683dec18fd2d3b64084fbb038b377f09d11b44b48351483f8e3b379aa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BRNN (Bayesian regularized neural network)
  - CMDN
  - RDKit
  - XGBoost
  - scikit-learn Random Forest
  edam_operation: http://edamontology.org/operation_0366
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# regression-error-metrics-evaluation

## Summary

Evaluates and compares regression models for retention-time (RT) prediction of xenobiotic metabolites using RMSE, R2, and MAE on a held-out test split, as done for XGBoost/RandomForest/BRNN models trained on the 1,021-pesticide LC-HRMS library.

## When to use

Use when trained regressors (e.g., RT prediction from molecular descriptors) must be validated on a held-out test set and compared via RMSE, R2, MAE, and an error-within-threshold fraction.

## When NOT to use

- The prediction problem is classification (e.g., metabolite detected/not-detected) rather than RT regression — use classification metrics instead
- Authentic analytical/stable-isotopic-labeled standards are available for the metabolites, so direct standard verification rather than in silico RT prediction is the appropriate validation channel
- There are no experimentally measured RTs for any compounds, so neither training nor held-out evaluation is possible

## Inputs

- Feature matrix of RDKit molecular descriptors computed from SMILES/InChIKey structures of the 1,021-pesticide library
- Experimentally measured LC-HRMS retention times (minutes) for the training compounds
- Trained regressor objects (XGBoost, scikit-learn Random Forest, BRNN) and their held-out 20% test-split predictions

## Outputs

- rt_prediction_metrics.csv with RMSE, R2, and MAE per model (XGB, RF, BRNN)
- Fraction of XGBoost test errors within 1 min of the measured RT (target: 95.1%)
- Ranked model comparison supporting selection of the best RT predictor

## How to apply

Split the descriptor-RT matrix (RDKit descriptors computed from SMILES/InChIKey of the 1,021-pesticide library) into 80:20 train/test sets stratified on the RT distribution. Train each candidate regressor — here XGBoost, scikit-learn Random Forest, and a Bayesian regularized neural network (BRNN) — with fixed hyperparameters for reproducibility, then predict RTs on the held-out 20%. Compute RMSE, R2, and MAE per model on the test split, plus the fraction of best-model errors falling within 1 min of the measured LC-HRMS RT. Select the model with the lowest error profile (XGBoost won: RMSE 0.89 min, R2 0.79, MAE 0.60 min, 95.1% of errors within 1 min) and write all metrics to a single comparison table (rt_prediction_metrics.csv).

## Related tools

- **XGBoost** (Best-performing regressor whose test-split RT predictions are scored with RMSE/R2/MAE and the 1-min error-fraction criterion)
- **scikit-learn Random Forest** (Comparison regressor trained on the same 80:20 split and scored with the same error metrics)
- **BRNN (Bayesian regularized neural network)** (Third comparison regressor evaluated alongside XGBoost and Random Forest)
- **RDKit** (Computes molecular descriptors from SMILES/InChIKey strings to form the regression feature matrix)

## Examples

```
from sklearn.metrics import mean_squared_error, r2_score, mean_absolute_error; metrics = {'RMSE': mean_squared_error(y_test, y_pred)**0.5, 'R2': r2_score(y_test, y_pred), 'MAE': mean_absolute_error(y_test, y_pred)}
```

## Evaluation signals

- XGBoost test-split metrics reproduce the reported values: RMSE = 0.89 min, R2 = 0.79, MAE = 0.60 min
- Fraction of XGBoost errors within 1 min of the measured RT reaches 95.1%
- Train/test partition is 80:20 and stratified on the RT distribution, not a random unstratified split
- All three models' metrics are written together to rt_prediction_metrics.csv for direct comparison
- Each model's hyperparameters are kept fixed (default or tuned) so the metric comparison is reproducible

## Limitations

- An R2 of 0.79 leaves roughly 21% of RT variance unexplained, so predicted RTs are supportive validation evidence rather than definitive structural confirmation alongside standard verification, murine studies, and time-course logistic modeling
- Metric values are tied to the specific LC-HRMS chromatographic conditions of the pesticide library and may not transfer to other columns or gradients
- The article's supplementary curation (CMDN-annotated PMDB, N = 492) indicates predicted-RT evidence still requires downstream manual curation before acceptance

## Evidence

- [results] Among three trained models (XGB, RF, BRNN), XGBoost achieved the best RT prediction performance on the pesticide library, with an RMSE of 0.89 min, R2 = 0.79, and MAE = 0.60 min.: [evidence span withheld — anchor status: not_found]
