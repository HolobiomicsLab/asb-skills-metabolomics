---
name: mb-vip-feature-importance-scoring
description: Use when a fitted MamsiPls MB-PLS model (e.g., on HPOS/LPOS/LNEG LC-MS
  feature tables) needs per-feature importance scores and empirical p-values to identify
  which predictors drive the predicted outcome.
when_to_use_negative:
- You need per-block (not per-feature) contribution estimates — use MamsiPls.block_importance
  instead of mb_vip
- You only need model performance assessment on a held-out test set — use evaluate_class_model/evaluate_regression_model
  rather than VIP-based feature scoring
- The fitted model is a single-block PLS where multiblock VIP is unnecessary; standard
  VIP suffices
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: mamsi
  role: Provides the MamsiPls class whose mb_vip and mb_vip_permtest methods compute
    MB-VIP scores and permutation-based empirical p-values
  repo: https://github.com/kopeckylukas/py-mamsi
- name: mbpls 1.0.4
  role: Underlying multiblock PLS algorithm (NIPALS) on which MB-VIP is computed
- name: pandas
  role: Loads and prefix-labels the assay feature tables and assembles the per-feature
    p-value output table
- name: numpy
  role: Masks features by p-value threshold (np.where(p_vals < 0.01)) to select the
    significant set
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:5d7155d981453c2279c569147bea739e3f1011cb148ff84f4f1e89bd1a14e76b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kopeckylukas/py-mamsi
  tools:
  - mamsi
  - matplotlib
  - mbpls
  - mbpls 1.0.4
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# mb-vip-feature-importance-scoring

## Summary

Scores predictor importance in a fitted multiblock PLS (MB-PLS) model using the multiblock variable importance in projection (MB-VIP) metric, then attaches empirical per-feature p-values via a permutation test on the outcome variable. This is the MAMSI framework's mechanism for selecting statistically significant features from multi-assay LC-MS metabolomics data.

## When to use

Use when a fitted MamsiPls MB-PLS model (e.g., on HPOS/LPOS/LNEG LC-MS feature tables) needs per-feature importance scores and empirical p-values to identify which predictors drive the predicted outcome.

## When NOT to use

- You need per-block (not per-feature) contribution estimates — use MamsiPls.block_importance instead of mb_vip
- You only need model performance assessment on a held-out test set — use evaluate_class_model/evaluate_regression_model rather than VIP-based feature scoring
- The fitted model is a single-block PLS where multiblock VIP is unnecessary; standard VIP suffices

## Inputs

- Fitted MamsiPls MB-PLS model (fitted on assay blocks with NIPALS, unit-variance standardization, cross-validation-estimated n_components, e.g., 6 LVs)
- Three LC-MS assay feature tables as pandas DataFrames with assay prefixes (alz_hpos.csv -> HPOS_, alz_lpos.csv -> LPOS_, alz_lneg.csv -> LNEG_)
- Numeric-encoded binary outcome variable (e.g., metadata['Gender'] encoded 1 = Female, 0 = Male)

## Outputs

- Array of MB-VIP scores, one entry per feature across all concatenated blocks
- Series/array of empirical p-values per feature (p = count of null VIP >= observed / n_permutations)
- Null MB-VIP score distributions from the permutation test (return_scores=True)
- Per-feature table (e.g., mb_vip_pvalues.csv) of observed MB-VIP scores and empirical p-values
- Statistically significant feature subset selected at p < 0.01

## How to apply

Fit the MB-PLS model with MamsiPls (NIPALS method, unit-variance standardization) on prefixed assay blocks such as HPOS_, LPOS_, and LNEG_ feature tables, using the number of latent variables estimated by cross-validation (6 LVs in the paper's Alzheimer's example, estimated via estimate_lv with metric='auc'). Compute multi-block VIP scores for all features with MamsiPls.mb_vip (get_scores=True, optionally plot=True). Then run MamsiPls.mb_vip_permtest on the data with n_permutations=10000, return_scores=True, and n_jobs=-1 to permute the numeric outcome (e.g., biological sex encoded as 1 for female, 0 for male), refit the model per permutation, and count null MB-VIP trials exceeding the observed statistic (p = count of null VIP >= observed / n_permutations). Assemble a per-feature table of observed MB-VIP scores and empirical p-values (e.g., mb_vip_pvalues.csv) and select significant features with the paper's threshold p < 0.01.

## Related tools

- **mamsi** (Provides the MamsiPls class whose mb_vip and mb_vip_permtest methods compute MB-VIP scores and permutation-based empirical p-values) — https://github.com/kopeckylukas/py-mamsi
- **mbpls 1.0.4** (Underlying multiblock PLS algorithm (NIPALS) on which MB-VIP is computed)
- **pandas** (Loads and prefix-labels the assay feature tables and assembles the per-feature p-value output table)
- **numpy** (Masks features by p-value threshold (np.where(p_vals < 0.01)) to select the significant set)

## Examples

```
p_vals, null_vip = mamsipls.mb_vip_permtest([hpos, lpos, lneg], y, n_permutations=10000, return_scores=True, n_jobs=-1)
```

## Evaluation signals

- The MB-VIP array has exactly one entry per feature across all concatenated assay blocks
- Every empirical p-value lies in [0, 1]
- p-values equal the count of null-permutation MB-VIP scores >= the observed statistic divided by n_permutations (10000)
- Features with p < 0.01 constitute the statistically significant set, matching the paper's selection threshold
- The returned null_vip object contains the null MB-VIP distributions when return_scores=True

## Limitations

- The permutation test is computationally expensive (10000 model refits); the tutorials repository provides sample_code/permtest_hpc for running it on a computer cluster (configuration relevant to Imperial HPC)
- MB-VIP importance is specific to the fitted MB-PLS model and its estimated number of latent variables (6 LVs in the paper); changing LV count changes the scores and p-values
- The framework was tested on metabolomics phenotyping data; the README notes it should be usable with other LC-MS data but was not validated there

## Evidence

- [abstract] MB-VIP with permutation testing: "We employ a multiblock-partial least-squares model (MB-PLS) coupled with multiblock variable importance in projection to estimate the importance of predictors to the outcome variable."
- [methods] permtest invocation: "p_vals, null_vip = mamsipls.mb_vip_permtest([hpos, lpos, lneg], y, n_permutations=10000, return_scores=True)"
- [methods] p-value threshold selection: "mask = np.where(p_vals < 0.01)"
- [readme] README feature list: "Estimation of statistically significant features (variables) using MB-VIP and permutation testing."
- [readme] outcome encoding: "The PLS algorithm requires the response variable to be numeric."
