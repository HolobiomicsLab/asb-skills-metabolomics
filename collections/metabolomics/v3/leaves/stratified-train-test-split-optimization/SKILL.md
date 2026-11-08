---
name: stratified-train-test-split-optimization
description: Use when authentic analytical standards are unavailable for CMDN-annotated
  metabolites and chromatographic plausibility must be validated by predicting LC-HRMS
  retention times from molecular structure (SMILES/InChIKey).
when_to_use_negative:
- Authentic analytical or stable-isotopic-labeled standards are available for the
  metabolites, so standard verification rather than in silico RT prediction should
  be used for validation
- The compounds of interest have no experimentally measured LC-HRMS retention times,
  so no labeled training set exists to fit or evaluate the regressors
- The goal is MS2-level annotation (spectral similarity) rather than chromatographic-behavior
  validation, which this RT-only regression does not address
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0154
- http://edamontology.org/topic_3520
tools:
- name: XGBoost
  role: Best-performing RT regression model on the pesticide library test split (RMSE
    0.89 min, R2 0.79, MAE 0.60 min)
- name: scikit-learn Random Forest
  role: Alternative RT regressor trained on the same descriptor matrix for model comparison
- name: BRNN (Bayesian regularized neural network)
  role: Third RT regressor benchmark against tree-based models
- name: LC-HRMS
  role: Source of the experimentally measured retention times used as regression targets
- name: CMDN
  role: Produces the CMDN-annotated metabolite structures whose chromatographic behavior
    the RT model validates
  repo: https://github.com/LeaveMeNotTonight/CMDN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:c0e0d6f3e9fe224b6bb71d491baa1ede954bca45e903ab59f759be6b8817ed5a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/LeaveMeNotTonight/CMDN
  tools:
  - BRNN (Bayesian regularized neural network)
  - CMDN
  - LC-HRMS
  - XGBoost
  - scikit-learn Random Forest
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0154
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# stratified-train-test-split-optimization

## Summary

Builds retention-time (RT) prediction models (XGBoost, Random Forest, Bayesian regularized neural network) on the 1,021-pesticide library using RDKit molecular descriptors, and applies an 80:20 train/test split stratified on the RT distribution so that model selection is reproducible and comparable across regressors.

## When to use

Use when authentic analytical standards are unavailable for CMDN-annotated metabolites and chromatographic plausibility must be validated by predicting LC-HRMS retention times from molecular structure (SMILES/InChIKey).

## When NOT to use

- Authentic analytical or stable-isotopic-labeled standards are available for the metabolites, so standard verification rather than in silico RT prediction should be used for validation
- The compounds of interest have no experimentally measured LC-HRMS retention times, so no labeled training set exists to fit or evaluate the regressors
- The goal is MS2-level annotation (spectral similarity) rather than chromatographic-behavior validation, which this RT-only regression does not address

## Inputs

- Pesticide library (1,021 compounds) with SMILES/InChIKey structure strings and experimentally measured LC-HRMS retention times
- RDKit molecular descriptor feature matrix computed from the structure strings

## Outputs

- rt_prediction_metrics.csv with RMSE, R2 and MAE per model (XGB, RF, BRNN) on the 20% test split
- Trained RT regression models (XGBoost, Random Forest, BRNN) applicable to CMDN-annotated metabolites lacking standards
- Fraction of XGBoost RT prediction errors within 1 min of the measured RT (reported: 95.1%)

## How to apply

Load the 1,021-pesticide library containing SMILES/InChIKey structures and their experimentally measured LC-HRMS retention times, then compute RDKit molecular descriptors from the structure strings to form the feature matrix for RT regression. Partition the descriptor-RT dataset into an 80:20 training and test split, stratifying on the RT distribution so the held-out set spans the chromatographic RT range rather than being randomly skewed. Train three regressors on the training split — XGBoost, a scikit-learn Random Forest, and a Bayesian regularized neural network (BRNN) — keeping each model's default or tuned hyperparameters fixed for reproducibility. Predict RTs for the held-out 20% test split, compute RMSE, R2 and MAE per model, and additionally compute the fraction of XGBoost errors falling within 1 min of the measured RT. Select the best model by these metrics; in this study XGBoost won with RMSE = 0.89 min, R2 = 0.79, MAE = 0.60 min, and 95.1% of its errors within 1 min, and the chosen model is then applied to score CMDN-annotated pesticide metabolites.

## Related tools

- **XGBoost** (Best-performing RT regression model on the pesticide library test split (RMSE 0.89 min, R2 0.79, MAE 0.60 min))
- **scikit-learn Random Forest** (Alternative RT regressor trained on the same descriptor matrix for model comparison)
- **BRNN (Bayesian regularized neural network)** (Third RT regressor benchmark against tree-based models)
- **LC-HRMS** (Source of the experimentally measured retention times used as regression targets)
- **CMDN** (Produces the CMDN-annotated metabolite structures whose chromatographic behavior the RT model validates) — https://github.com/LeaveMeNotTonight/CMDN

## Examples

```
Rscript cmdn_pipeline.R  # scripts and raw data are located from the comment lines in the LeaveMeNotTonight/CMDN repository; RT model evaluation writes rt_prediction_metrics.csv
```

## Evaluation signals

- XGBoost RMSE on the 20% stratified test split matches the reported 0.89 min (R2 = 0.79, MAE = 0.60 min)
- Fraction of XGBoost test-split errors within 1 min of measured RT reaches 95.1%
- The stratified split covers the RT distribution such that the held-out 20% is representative across the chromatographic range, not clustered at one RT extreme
- Metrics for all three models (XGB, RF, BRNN) are written to rt_prediction_metrics.csv with hyperparameters fixed and reproducible
- Descriptors are computed consistently from SMILES/InChIKey strings for both training pesticides and CMDN-annotated metabolites at prediction time

## Limitations

- RT prediction is an in silico validation channel only; the paper treats it as one of multiple validation channels alongside standard verification, time-course logistic modeling and murine studies, not as direct structural proof
- Stratification is on the RT distribution of the 1,021-pesticide library; metabolites far outside that chemical/RT space may be extrapolation cases for the model
- Manual curation in the source study (e.g. the N = 492 manually curated CMDN-annotated PMDB subset) still supplements automated validation

## Evidence

- [results] XGBoost achieved the best RT prediction performance: [evidence span withheld — anchor status: not_found]
