---
name: mb-vip-permutation-significance-testing
description: Use when an MB-PLS model (e.g. MamsiPls on multi-assay serum LC-MS intensity
  tables) has been fitted and you need per-feature statistical significance for feature
  selection before structural/correlation clustering, rather than relying on raw VIP
  scores alone.
when_to_use_negative:
- The goal is model prediction performance evaluation (use evaluate_regression_model
  / evaluate_class_model on the held-out test set instead of feature significance
  testing)
- Data are single-assay or non-LC-MS where a multi-block VIP formulation and structural-cluster
  interpretation (rt_win, ppm adduct/isotopologue grouping) is not applicable
- The model's number of latent variables has not yet been validated (run estimate_lv
  with k-fold cross-validation first, otherwise permuted VIPs reflect an ill-sized
  model)
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: MAMSI
  role: Python framework providing the MamsiPls class whose mb_vip and mb_vip_permtest
    methods compute MB-VIP scores and permutation-based empirical p-values
  repo: https://github.com/kopeckylukas/py-mamsi
- name: mamsi
  role: Installable PyPI package (pip install mamsi) exposing mamsi.mamsi_pls.MamsiPls
    used for the permutation testing
  repo: https://github.com/kopeckylukas/py-mamsi
- name: mbpls
  role: Pinned dependency (mbpls==1.0.4) underlying the multiblock PLS algorithm on
    which MB-VIP is computed
- name: numpy
  role: Used to mask features by p-value threshold, e.g. np.where(p_vals < 0.01),
    to select significant columns
- name: joblib
  role: Dependency enabling n_jobs=-1 parallelization of the permutation loop
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:b6db44199c731d6b7652916e40d0289f35ac8ee5dd6e5538baeba85632e4e489
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kopeckylukas/py-mamsi
  tools:
  - MAMSI
  - Python
  - joblib
  - mamsi
  - matplotlib
  - mbpls
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# mb-vip-permutation-significance-testing

## Summary

Assigns empirical p-values to every feature in a multi-block PLS (MB-PLS) model by permuting the outcome variable and recomputing Multi-Block Variable Importance in Projection (MB-VIP) scores, so that statistically significant cross-assay LC-MS features can be selected at a controlled p-value threshold (e.g. p < 0.01).

## When to use

Use when an MB-PLS model (e.g. MamsiPls on multi-assay serum LC-MS intensity tables) has been fitted and you need per-feature statistical significance for feature selection before structural/correlation clustering, rather than relying on raw VIP scores alone.

## When NOT to use

- The goal is model prediction performance evaluation (use evaluate_regression_model / evaluate_class_model on the held-out test set instead of feature significance testing)
- Data are single-assay or non-LC-MS where a multi-block VIP formulation and structural-cluster interpretation (rt_win, ppm adduct/isotopologue grouping) is not applicable
- The model's number of latent variables has not yet been validated (run estimate_lv with k-fold cross-validation first, otherwise permuted VIPs reflect an ill-sized model)

## Inputs

- Fitted MamsiPls MB-PLS model (regression or classification) with estimated number of latent variables
- List of assay-block feature intensity DataFrames (e.g. HPOS/LPOS/LNEG LC-MS tables with assay-prefixed column names)
- Outcome variable vector y (e.g. continuous serum bilirubin levels or encoded phenotype)
- n_permutations (>=10000 recommended, ideally >100000), n_jobs, return_scores parameters

## Outputs

- Empirical p-value array for every feature across all blocks
- Null MB-VIP score distributions (when return_scores=True)
- Selected significant feature table (columns at p < 0.01) ready for MamsiStructSearch.load_lcms()

## How to apply

After fitting the MB-PLS model with MamsiPls (NIPALS algorithm, standardized blocks) and choosing the number of latent variables via estimate_lv() cross-validation (metric='q2' for regression tasks like predicting bilirubin, metric='auc' for classification), call mb_vip(plot=True) to compute MB-VIP scores for all features across all assay blocks. Then run mb_vip_permtest() with the full data blocks and outcome, setting n_permutations=10000 as a minimum (ideally >100000) and n_jobs=-1 to parallelize; the method permutes the Y outcome variable and recomputes null MB-VIP distributions to derive an empirical p-value per feature. Select significant features with a mask such as np.where(p_vals < 0.01), optionally with multiple-testing correction, and concatenate the selected columns across blocks (with assay prefixes like HPOS_, LPOS_, LNEG_) for downstream MamsiStructSearch analysis. The permutation approach is preferred over raw VIP cut-offs because it yields a per-feature null distribution calibrated to the actual model and outcome.

## Related tools

- **MAMSI** (Python framework providing the MamsiPls class whose mb_vip and mb_vip_permtest methods compute MB-VIP scores and permutation-based empirical p-values) — https://github.com/kopeckylukas/py-mamsi
- **mamsi** (Installable PyPI package (pip install mamsi) exposing mamsi.mamsi_pls.MamsiPls used for the permutation testing) — https://github.com/kopeckylukas/py-mamsi
- **mbpls** (Pinned dependency (mbpls==1.0.4) underlying the multiblock PLS algorithm on which MB-VIP is computed)
- **numpy** (Used to mask features by p-value threshold, e.g. np.where(p_vals < 0.01), to select significant columns)
- **joblib** (Dependency enabling n_jobs=-1 parallelization of the permutation loop)

## Examples

```
p_vals, null_vip = mamsipls.mb_vip_permtest([hpos, lpos, lneg], y, n_permutations=10000, return_scores=True); selected = x.iloc[:, np.where(p_vals < 0.01)[0]]
```

## Evaluation signals

- Selected feature count should be stable when n_permutations is increased from 10000 toward >100000 (features near threshold may flip, but the overall signature should hold)
- For the MY Diabetes bilirubin regression reference run, feature selection at p < 0.01 should recover the reported 18-feature signature (two structural groups and three singletons after MamsiStructSearch)
- Empirical p-values should be near-uniform under complete permutation of the outcome; features selected at p < 0.01 should also rank high on the observed mb_vip scores
- Signs of association of the selected (putatively annotated) biomarkers should agree with previously reported literature results, as reported in the article

## Limitations

- 10000 permutations is the documented default and gives coarse resolution for small empirical p-values; the workflow recommends >100000 permutations (the tutorials provide an HPC sample_code/permtest_hpc configuration for running this at scale)
- Permutation testing is computationally intensive (n_jobs=-1 and cluster/HPC execution may be needed for full multi-assay feature sets)
- Annotation of untargeted LC-MS features downstream is only supported for assays analysed by the National Phenome Centre; significance alone does not guarantee putative annotation

## Evidence

- [methods] Permutation testing to obtain empirical p-values per feature: [evidence span withheld — anchor status: not_found]
- [methods] p-value threshold selection code: "p_vals, null_vip = mamsipls.mb_vip_permtest([hpos, lpos, lneg], y, n_permutations=10000, return_scores=True)"
- [methods] Threshold selection snippet: "mask = np.where(p_vals < 0.01)
selected = x.iloc[:, mask[0]]"
- [readme] README description of significance estimation: "Estimation of statistically significant features (variables) using MB-VIP and permutation testing."
- [methods] Task-level recommendation on permutation count: [evidence span withheld — anchor status: not_found]
