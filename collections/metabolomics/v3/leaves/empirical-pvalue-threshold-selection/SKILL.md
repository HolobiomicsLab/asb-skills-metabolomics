---
name: empirical-pvalue-threshold-selection
description: Use when empirical p-values from mb_vip_permtest are available for a
  multi-assay LC-MS feature matrix (HPOS_/LPOS_/LNEG_ blocks) and significant predictors
  must be selected at a p-value threshold for counting, comparison against univariate
  models, or downstream structural clustering.
when_to_use_negative:
- When only a single-assay univariate analysis is being performed and no MB-PLS/MB-VIP
  model has been fitted — empirical p-values from mb_vip_permtest do not exist
- When the input is a curated/targeted feature list rather than untargeted multi-assay
  LC-MS feature tables requiring statistical selection
- When the goal is latent-variable estimation or model evaluation (use estimate_lv
  / evaluate_class_model) rather than significant-feature selection
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mamsi
  role: Provides MamsiPls.mb_vip_permtest, which generates the empirical p-values
    that the thresholding selection is applied to
  repo: https://github.com/kopeckylukas/py-mamsi
- name: numpy
  role: Supplies the np.where mask pattern used to threshold p-values and index the
    concatenated feature matrix
- name: pandas
  role: Loads and concatenates the HPOS_/LPOS_/LNEG_ assay CSV tables into the feature
    matrix sliced by the significance mask
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:f2edd1a3cae40d18d3c184cd9ee0df54efb3d42684505c8e0fc823439d191c5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kopeckylukas/py-mamsi
  tools:
  - Python
  - mamsi
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# empirical-pvalue-threshold-selection

## Summary

Selects statistically significant features from an MB-PLS model by thresholding empirical p-values obtained from the MAMSI MB-VIP permutation test, using a p < 0.01 mask on the concatenated multi-assay feature matrix (with optional alpha = 0.05 or Bonferroni-corrected thresholds) to quantify per-assay significant-feature reductions.

## When to use

Use when empirical p-values from mb_vip_permtest are available for a multi-assay LC-MS feature matrix (HPOS_/LPOS_/LNEG_ blocks) and significant predictors must be selected at a p-value threshold for counting, comparison against univariate models, or downstream structural clustering.

## When NOT to use

- When only a single-assay univariate analysis is being performed and no MB-PLS/MB-VIP model has been fitted — empirical p-values from mb_vip_permtest do not exist
- When the input is a curated/targeted feature list rather than untargeted multi-assay LC-MS feature tables requiring statistical selection
- When the goal is latent-variable estimation or model evaluation (use estimate_lv / evaluate_class_model) rather than significant-feature selection

## Inputs

- Empirical p-values per feature from mamsipls.mb_vip_permtest (n_permutations=10000)
- Three LC-MS assay feature tables as CSV: alz_hpos.csv (HPOS_ prefixed), alz_lpos.csv (LPOS_ prefixed), alz_lneg.csv (LNEG_ prefixed)
- Numeric outcome variable y (e.g. Gender encoded 1=Female, 0=Male from alz_metadata.csv)
- Fitted MamsiPls MB-PLS model

## Outputs

- Selected significant feature matrix subset (selected = x.iloc[:, mask[0]])
- Per-assay significant-feature counts partitioned by HPOS_/LPOS_/LNEG_ column prefixes
- Reduction percentages of MB-PLS vs univariate significant predictors per experiment (validated against Table 2: 83%, >92%, >92%, 94%)
- significant_features_table.csv with per-experiment counts

## How to apply

Fit the MamsiPls MB-PLS model on the three LC-MS assay blocks and run mb_vip_permtest with n_permutations=10000 to obtain per-feature empirical p-values by permuting the Y outcome variable. Concatenate the prefixed assay tables (alz_hpos.csv, alz_lpos.csv, alz_lneg.csv) into a single feature matrix with pandas, then apply the numpy mask pattern np.where(p_vals < 0.01) and select columns via x.iloc[:, mask[0]]. Optionally repeat the selection at alpha = 0.05 and at a Bonferroni-corrected threshold (alpha divided by the number of features tested per experiment), since the paper notes multiple testing correction can be applied to adjust the threshold. Map selected feature indices back to their HPOS_/LPOS_/LNEG_ column prefixes to count significant predictors per assay block, and compare MB-PLS counts against univariate-analysis counts to compute the reduction reported in Table 2 (83% for experiment 1, >92% for experiments 2 and 3, 94% for experiment 4), writing per-experiment counts to significant_features_table.csv.

## Related tools

- **mamsi** (Provides MamsiPls.mb_vip_permtest, which generates the empirical p-values that the thresholding selection is applied to) — https://github.com/kopeckylukas/py-mamsi
- **numpy** (Supplies the np.where mask pattern used to threshold p-values and index the concatenated feature matrix)
- **pandas** (Loads and concatenates the HPOS_/LPOS_/LNEG_ assay CSV tables into the feature matrix sliced by the significance mask)

## Examples

```
p_vals, null_vip = mamsipls.mb_vip_permtest([hpos, lpos, lneg], y, n_permutations=10000, return_scores=True); mask = np.where(p_vals < 0.01); selected = x.iloc[:, mask[0]]
```

## Evaluation signals

- Per-experiment significant-feature counts reproduce the Table 2 reduction values: 83% reduction for experiment 1, >92% for experiments 2 and 3, and 94% for experiment 4
- All empirical p-values fall in [0, 1] and the number of returned p-values equals the number of columns in the concatenated HPOS_+LPOS_+LNEG_ feature matrix
- Every selected column index maps to a feature whose name carries one of the HPOS_, LPOS_, or LNEG_ prefixes
- The Bonferroni-corrected selection (alpha divided by features tested per experiment) is a strict subset of the alpha = 0.05 selection
- Re-running the mask at alpha = 0.01 yields a stable subset of features suitable for downstream MamsiStructSearch clustering

## Limitations

- The paper notes multiple testing correction 'can be applied to adjust the p-value threshold' but does not mandate it, so threshold choice (0.01, 0.05, Bonferroni) directly changes the selected feature set and downstream clustering
- The permutation test is computationally expensive at n_permutations=10000; the tutorials repository provides sample_code/permtest_hpc guidance for running it on a computer cluster (configuration relevant for Imperial HPC)
- Reduction percentages are specific to the four MAMSI experiments and the AddNeuroMed sample data; they will not transfer to other datasets and must be recomputed

## Evidence

- [methods] permutation p-value mask: "mask = np.where(p_vals < 0.01)
selected = x.iloc[:, mask[0]]"
- [methods] permtest parameters: "p_vals, null_vip = mamsipls.mb_vip_permtest([hpos, lpos, lneg], y, n_permutations=10000, return_scores=True)"
- [methods] permtest definition: "Calculate empirical p-values for each feature by permuting the Y outcome variable"
- [readme] README feature description: "Estimation of statistically significant features (variables) using MB-VIP and permutation testing."
- [full_text] task validation finding: [evidence span withheld — anchor status: not_found]
