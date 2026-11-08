---
name: bayesian-neural-network-regression
description: Use when you have a compound library with SMILES/InChIKey structures
  and experimentally measured LC-HRMS retention times, and need a trained regressor
  to predict RTs for metabolites lacking authentic standards.
when_to_use_negative:
- No experimentally measured RTs are available for training — the regressor requires
  a labeled descriptor–RT dataset and cannot validate metabolites from structures
  alone.
- The goal is structural annotation or biotransformation discovery of xenobiotic metabolites
  — that is CMDN's top-down annotation task (DMS/RCS/SSN layers), not RT regression.
- The compounds fall outside the pesticide-like chemical space used to fit the model,
  where descriptor-based RT predictions would not be calibrated.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: BRNN (Bayesian regularized neural network)
  role: One of three RT regressors trained on RDKit descriptors and benchmarked on
    the 20% test split
- name: XGBoost
  role: Primary RT regressor benchmark that defines the reported best metrics (RMSE
    0.89 min, R2 0.79, MAE 0.60 min) against which BRNN is compared
- name: CMDN
  role: Produces the CMDN-annotated metabolites whose chromatographic behavior is
    validated by the RT prediction models
  repo: github.com__LeaveMeNotTonight__CMDN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.est.5c08558
    title: Reaction-Guided Metabolomics Accelerates High-Throughput Annotation of
      Xenobiotic Metabolites for Human Exposome
  source_content_hash: sha256:e553539184a94b4cf7fcf531ff98edb26412512cf21aaee2e5a7a5f8568f7447
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - BRNN (Bayesian regularized neural network)
  - CMDN
  - XGBoost
  - scikit-learn Random Forest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# bayesian-neural-network-regression

## Summary

Trains a Bayesian regularized neural network (BRNN) to regress LC retention time (RT) from molecular descriptors computed on SMILES/InChIKey structures, serving as one of three benchmark models (with XGBoost and Random Forest) for validating chromatographic behavior of CMDN-annotated pesticide metabolites when authentic standards are unavailable.

## When to use

Use when you have a compound library with SMILES/InChIKey structures and experimentally measured LC-HRMS retention times, and need a trained regressor to predict RTs for metabolites lacking authentic standards.

## When NOT to use

- No experimentally measured RTs are available for training — the regressor requires a labeled descriptor–RT dataset and cannot validate metabolites from structures alone.
- The goal is structural annotation or biotransformation discovery of xenobiotic metabolites — that is CMDN's top-down annotation task (DMS/RCS/SSN layers), not RT regression.
- The compounds fall outside the pesticide-like chemical space used to fit the model, where descriptor-based RT predictions would not be calibrated.

## Inputs

- Pesticide library (1,021 compounds) with SMILES/InChIKey structures
- Experimentally measured LC-HRMS retention times for the training compounds
- RDKit molecular descriptor feature matrix computed from the structure strings

## Outputs

- Trained BRNN RT regression model
- Predicted retention times for the held-out 20% test split
- Test-split RMSE, R2, and MAE for BRNN alongside XGBoost and Random Forest, written to rt_prediction_metrics.csv

## How to apply

Load the 1,021-pesticide library with SMILES/InChIKey structures and experimentally measured LC-HRMS retention times, then compute molecular descriptors (e.g., RDKit descriptors) from the structure strings to build the feature matrix for RT regression. Partition the descriptor–RT dataset into an 80:20 train/test split (stratified on the RT distribution). Train the BRNN on the training split alongside XGBoost and scikit-learn Random Forest, keeping hyperparameters fixed for reproducibility, and predict RTs only for the held-out 20% test split. Report RMSE, R2, and MAE per model on the test split; in the source study XGBoost achieved the best performance (RMSE 0.89 min, R2 = 0.79, MAE = 0.60 min), so the BRNN is interpreted as a comparison baseline rather than the primary RT predictor. The rationale is multichannel validation: the trained RT models let CMDN-annotated metabolites be checked chromatographically in silico where stable-isotope-labeled or analytical standards are not available.

## Related tools

- **BRNN (Bayesian regularized neural network)** (One of three RT regressors trained on RDKit descriptors and benchmarked on the 20% test split)
- **XGBoost** (Primary RT regressor benchmark that defines the reported best metrics (RMSE 0.89 min, R2 0.79, MAE 0.60 min) against which BRNN is compared)
- **CMDN** (Produces the CMDN-annotated metabolites whose chromatographic behavior is validated by the RT prediction models) — github.com__LeaveMeNotTonight__CMDN

## Evaluation signals

- The descriptor–RT dataset is split 80:20 stratified on the RT distribution, and predictions are made only on the held-out test split.
- RMSE, R2, and MAE are computed on the test split for all three models (BRNN, XGBoost, RF) and written to rt_prediction_metrics.csv.
- Reproduced XGBoost metrics match the reported values (RMSE 0.89 min, R2 = 0.79, MAE = 0.60 min) and the fraction of XGBoost errors within 1 min of measured RT reaches 95.1%.
- BRNN predictions are compared head-to-head with XGBoost and Random Forest on identical splits, and BRNN should not exceed the XGBoost error profile if the pipeline is reproduced faithfully.
- Hyperparameters for all three regressors are fixed and documented so the run is reproducible.

## Limitations

- BRNN was not the top performer in the source study — XGBoost achieved the best RT prediction performance — so BRNN serves mainly as a benchmark comparison.
- The RT model requires authentic measured RTs for training; it predicts chromatographic behavior for compounds whose standards are unavailable only after training on the labeled library.
- Predicted RTs provide in silico (not direct analytical) evidence; the paper emphasizes that in silico approaches 'can provide partial insights', and full validation additionally uses analytical/stable-isotopic-labeled standards, time-course logistic modeling, and in vivo mice studies.

## Evidence

- [methods] Among three trained models (XGB, RF, BRNN), XGBoost achieved the best RT prediction performance on the pesticide library, with an RMSE of 0.89 min, R2 = 0.79, and MAE = 0.60 min.: [evidence span withheld — anchor status: not_found]
- [abstract] Multichannel validation including standard verification, retention time prediction, murine studies, and time-course logistic modeling: "Following multichannel validation, including standard verification, retention time prediction, murine studies, and time-course logistic modeling"
- [intro] Characterization of metabolites by accurate m/z, retention time (RT), and MS2 spectra: "Resulting metabolites were characterized by accurate m/z, retention time (RT), and MS2 spectra"
- [intro] Multiple-channel validation: analytical/stable-isotopic-labeled standards, time-course logistic modeling, in silico prediction and in vivo mice studies: "further validated them through multiple channels validation, including analytical/stable-isotopic-labeled standards, time-course logistic modeling, in silico"
- [readme] CMDN is an 'top-down' untargeted metabolomics-based MS data processing framework to enable high-throughput and automated annotation of reaction-derived xenobiotic metabolites.: "Compound metabolite discovery network (CMDN) is an “top-down” untargeted metabolomics-based MS data processing framework to enable high-throughput"
