---
name: permutation-testing-feature-significance
description: Use when you have a fitted MB-PLS model over multiple LC-MS assay blocks
  (e.g. HPOS/LPOS/LNEG) and need statistically defensible feature significance beyond
  raw MB-VIP magnitudes before biomarker annotation or structural search.
when_to_use_negative:
- You need held-out model performance (accuracy, recall, specificity, F1, AUC) rather
  than per-feature significance — use evaluate_class_model/evaluate_regression_model
  instead
- You have a single-assay feature table with no multi-block structure — a plain univariate
  or single-block VIP approach is appropriate; MB-VIP permutation testing is designed
  for interblock MB-PLS models
- Features are already annotated, targeted metabolite concentrations where standard
  univariate testing suffices rather than untargeted multi-assay feature significance
  estimation
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mamsi
  role: Provides MamsiPls.mb_vip_permtest to permute the Y outcome and compute empirical
    per-feature p-values from null MB-VIP distributions
  repo: https://github.com/kopeckylukas/py-mamsi
- name: numpy
  role: np.where(p_vals < 0.01) masking to select statistically significant feature
    indices
- name: pandas
  role: Concatenates the HPOS/LPOS/LNEG assay blocks (axis=1) and indexes significant
    columns via x.iloc[:, mask[0]]
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:0b484c9df779b0454791b3adc447fb7f2961cb438424fd7c1bad090345925b71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kopeckylukas/py-mamsi
  tools:
  - Python
  - joblib
  - mamsi
  - matplotlib
  - mbpls
  - networkx
  - numpy
  - pandas
  - pyvis
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# permutation-testing-feature-significance

## Summary

Estimate empirical per-feature p-values for multiassay LC-MS metabolomics features by permuting the outcome variable and recomputing MB-VIP scores with MamsiPls, then selecting significant predictors (p < 0.01) for downstream structural clustering.

## When to use

Use when you have a fitted MB-PLS model over multiple LC-MS assay blocks (e.g. HPOS/LPOS/LNEG) and need statistically defensible feature significance beyond raw MB-VIP magnitudes before biomarker annotation or structural search.

## When NOT to use

- You need held-out model performance (accuracy, recall, specificity, F1, AUC) rather than per-feature significance — use evaluate_class_model/evaluate_regression_model instead
- You have a single-assay feature table with no multi-block structure — a plain univariate or single-block VIP approach is appropriate; MB-VIP permutation testing is designed for interblock MB-PLS models
- Features are already annotated, targeted metabolite concentrations where standard univariate testing suffices rather than untargeted multi-assay feature significance estimation

## Inputs

- Fitted MamsiPls MB-PLS model (e.g. NIPALS, standardized blocks, LV count estimated via estimate_lv with metric='auc')
- List of assay feature matrices as pandas DataFrames with assay-prefixed feature columns (HPOS_, LPOS_, LNEG_)
- Numeric-encoded outcome variable y (e.g. binary 1/0 for classification, continuous for regression)

## Outputs

- Empirical p-value array (one p-value per concatenated feature across all assay blocks)
- Null MB-VIP score distributions (returned when return_scores=True)
- Subset of statistically significant features (p < 0.01) selected by boolean mask into the concatenated feature table

## How to apply

Fit a MamsiPls model on the concatenated assay matrices (column-prefixed HPOS_, LPOS_, LNEG_) with a numeric outcome, then call mb_vip_permtest with n_permutations=10000 and return_scores=True to obtain empirical p-values and the null MB-VIP distribution per feature. Permutation of the Y outcome variable destroys the feature–outcome association while preserving each feature's variance, yielding an empirical null for every predictor. Select significant features with a p < 0.01 threshold via np.where(p_vals < 0.01) and index into the concatenated feature table (x.iloc[:, mask[0]]). Because 10,000 permutations are costly, the tutorials provide sample_code/permtest_hpc for running the permutation test on a computer cluster. The selected features then feed MamsiStructSearch (rt_win=5, ppm=10) for isotopologue, adduct and cross-assay cluster assignment.

## Related tools

- **mamsi** (Provides MamsiPls.mb_vip_permtest to permute the Y outcome and compute empirical per-feature p-values from null MB-VIP distributions) — https://github.com/kopeckylukas/py-mamsi
- **numpy** (np.where(p_vals < 0.01) masking to select statistically significant feature indices)
- **pandas** (Concatenates the HPOS/LPOS/LNEG assay blocks (axis=1) and indexes significant columns via x.iloc[:, mask[0]])

## Examples

```
p_vals, null_vip = mamsipls.mb_vip_permtest([hpos, lpos, lneg], y, n_permutations=10000, return_scores=True)
```

## Evaluation signals

- p-value array length equals the number of concatenated features across all assay blocks (HPOS + LPOS + LNEG columns)
- mb_vip_permtest called with n_permutations=10000 and return_scores=True returns both p_vals and the null_vip score matrix
- Selected feature set is non-empty and retains assay block prefixes (HPOS_, LPOS_, LNEG_) when extracted via x.iloc[:, mask[0]]
- Features passing p < 0.01 should correspond to high MB-VIP predictors, and their structural clusters (isotopologue, adduct, cross-assay, correlation) are resolvable by MamsiStructSearch(rt_win=5, ppm=10)
- Re-running the permutation test should yield stable significance calls for strong biomarkers (p-values near 0), as in the reference workflow where significant cross-assay biomarkers agreed with literature-reported associations

## Limitations

- 10,000 permutations per feature are computationally expensive; the tutorials ship sample_code/permtest_hpc for running the permutation test on a computer cluster (e.g. Imperial HPC)
- The p < 0.01 significance threshold is a workflow convention; downstream annotation of untargeted LC-MS features is only supported for assays analysed by the National Phenome Centre (ROI files from peakPantheR)
- Empirical p-values depend on the fitted MB-PLS model; an under- or over-fitted model (wrong LV count) will distort both MB-VIP magnitudes and their permutation nulls

## Evidence

- [methods] Permutation testing to obtain empirical p-values per feature: "Calculate empirical p-values for each feature by permuting the Y outcome variable"
- [readme] Permutation test call with parameters: "p_vals, null_vip = mamsipls.mb_vip_permtest([hpos, lpos, lneg], y, n_permutations=10000, return_scores=True)"
- [methods] Significance threshold and feature selection: "mask = np.where(p_vals < 0.01)
selected = x.iloc[:, mask[0]]"
- [readme] Estimation of statistically significant features feature description: "Estimation of statistically significant features (variables) using MB-VIP and permutation testing."
