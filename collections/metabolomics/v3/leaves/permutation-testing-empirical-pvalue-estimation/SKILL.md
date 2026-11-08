---
name: permutation-testing-empirical-pvalue-estimation
description: Use when a fitted multiblock PLS model (e.g. MamsiPls on multi-assay
  LC-MS feature tables) needs statistically defensible per-feature significance, and
  MB-VIP scores alone cannot distinguish signal from chance.
when_to_use_negative:
- You only need block-level (not per-feature) contribution — use MamsiPls.block_importance
  instead of the permutation test
- The model has not yet been fitted and its number of latent variables estimated (estimate_lv)
  — permuting an unfitted or misspecified model yields meaningless nulls
- Data are not from a multi-assay LC-MS metabolomics design where MB-VIP is the intended
  importance metric — the null distribution is defined on MB-VIP statistics specifically
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: mamsi
  role: Provides MamsiPls.mb_vip and MamsiPls.mb_vip_permtest to compute observed
    MB-VIP scores and run the permutation test for empirical p-values
  repo: https://github.com/kopeckylukas/py-mamsi
- name: mbpls
  role: mbpls==1.0.4 underpins the MB-PLS model refit in each permutation trial (NIPALS
    method)
- name: numpy
  role: Used to mask features by p-value threshold (np.where(p_vals < 0.01)) when
    assembling the significant feature set
- name: pandas
  role: Loads the assay feature tables and assembles the per-feature p-value output
    table (mb_vip_pvalues.csv)
- name: joblib
  role: Parallelizes the 10000-permutation test (n_jobs=-1) as a MAMSI dependency
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:e8ce5a95a773f31495d58ce61a7cd3a617489da0d38b6e0f983708ea6f3a3a9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kopeckylukas/py-mamsi
  tools:
  - joblib
  - mamsi
  - matplotlib
  - mbpls
  - numpy
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# permutation-testing-empirical-pvalue-estimation

## Summary

Assigns empirical per-feature p-values to a fitted MB-PLS model by coupling multi-block variable importance in projection (MB-VIP) scores with a label-permutation null: the outcome variable is repeatedly permuted, the model refit, and null MB-VIP scores compared to the observed statistics.

## When to use

Use when a fitted multiblock PLS model (e.g. MamsiPls on multi-assay LC-MS feature tables) needs statistically defensible per-feature significance, and MB-VIP scores alone cannot distinguish signal from chance.

## When NOT to use

- You only need block-level (not per-feature) contribution — use MamsiPls.block_importance instead of the permutation test
- The model has not yet been fitted and its number of latent variables estimated (estimate_lv) — permuting an unfitted or misspecified model yields meaningless nulls
- Data are not from a multi-assay LC-MS metabolomics design where MB-VIP is the intended importance metric — the null distribution is defined on MB-VIP statistics specifically

## Inputs

- Fitted MamsiPls MB-PLS model (NIPALS, unit-variance standardized blocks, 6 latent variables)
- Three LC-MS assay feature tables as pandas DataFrames: alz_hpos.csv, alz_lpos.csv, alz_lneg.csv (assay-prefixed columns, e.g. HPOS_, LPOS_, LNEG_)
- Numeric-encoded binary outcome variable (e.g. Gender encoded 1 = female, 0 = male)
- Number of permutations (10000 in the paper)

## Outputs

- Empirical p-value array with one entry per feature (p = count of null VIP >= observed / n_permutations)
- Null MB-VIP score distribution (returned when return_scores=True)
- Per-feature table of observed MB-VIP scores and empirical p-values (mb_vip_pvalues.csv); features with p < 0.01 form the significant set

## How to apply

After fitting the MamsiPls model (NIPALS, unit-variance standardization, 6 latent variables estimated via cross-validation with metric='auc') and computing observed scores with MamsiPls.mb_vip(get_scores=True), run MamsiPls.mb_vip_permtest on the full dataset with n_permutations=10000, return_scores=True, and n_jobs=-1 so the outcome variable (e.g. binary biological sex encoded 1/0) is permuted, the MB-PLS model is refit each trial, and null MB-VIP scores are collected. The empirical p-value for each feature equals the count of permutation trials with null MB-VIP >= the observed statistic divided by n_permutations. Assemble a per-feature table of observed MB-VIP scores and empirical p-values (e.g. mb_vip_pvalues.csv), then select significant features with the paper's threshold p < 0.01. For computational tractability on large permutation counts, the tutorials repository provides sample_code/permtest_hpc for running the permtest on a computer cluster (e.g. Imperial HPC).

## Related tools

- **mamsi** (Provides MamsiPls.mb_vip and MamsiPls.mb_vip_permtest to compute observed MB-VIP scores and run the permutation test for empirical p-values) — https://github.com/kopeckylukas/py-mamsi
- **mbpls** (mbpls==1.0.4 underpins the MB-PLS model refit in each permutation trial (NIPALS method))
- **numpy** (Used to mask features by p-value threshold (np.where(p_vals < 0.01)) when assembling the significant feature set)
- **pandas** (Loads the assay feature tables and assembles the per-feature p-value output table (mb_vip_pvalues.csv))
- **joblib** (Parallelizes the 10000-permutation test (n_jobs=-1) as a MAMSI dependency)

## Examples

```
p_vals, null_vip = mamsipls.mb_vip_permtest([hpos, lpos, lneg], y, n_permutations=10000, return_scores=True, n_jobs=-1); mask = np.where(p_vals < 0.01)
```

## Evaluation signals

- Every returned p-value lies in [0, 1]
- The MB-VIP array has exactly one entry per feature across all blocks (HPOS + LPOS + LNEG columns)
- With n_permutations=10000 and return_scores=True, the null_vip object contains 10000 null MB-VIP score draws per feature
- Features with p < 0.01 reproduce the paper's statistically significant feature set, which feeds downstream structural clustering (MamsiStructSearch)
- p-values should be largest for features with small observed MB-VIP relative to their permutation null distribution

## Limitations

- The permutation test is computationally heavy: 10000 model refits typically require a compute cluster (the tutorials ship permtest_hpc sample code for Imperial HPC)
- Empirical p-values are bounded by 1/n_permutations (best achievable resolution is p = 0 at 10000 permutations)
- Permutation p-values are specific to the fitted model configuration (6 LVs, standardization, block structure); changing the model invalidates the null distribution

## Evidence

- [methods] Permutation testing to obtain empirical p-values per feature: [evidence span withheld — anchor status: not_found]
- [methods] permtest call signature: "p_vals, null_vip = mamsipls.mb_vip_permtest([hpos, lpos, lneg], y, n_permutations=10000, return_scores=True)"
- [methods] p-value threshold selection: "mask = np.where(p_vals < 0.01)
selected = x.iloc[:, mask[0]]"
- [methods] permtest definition from filter steps: "Calculate empirical p-values for each feature by permuting the Y outcome variable"
- [readme] README permtest usage: "or estimate empirical p-values for all features: 

```python
p_vals, null_vip = mamsipls.mb_vip_permtest([hpos, lpos, lneg], y, n_permutations=10000, return_scores=True)"
- [readme] HPC guidance for permutations: [evidence span withheld — anchor status: external_doc]
