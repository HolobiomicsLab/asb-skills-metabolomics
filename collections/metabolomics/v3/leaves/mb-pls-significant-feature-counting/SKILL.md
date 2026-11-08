---
name: mb-pls-significant-feature-counting
description: Use when you have fitted a multi-block PLS model over concatenated HPOS/LPOS/LNEG
  feature tables, obtained per-feature empirical p-values from an MB-VIP permutation
  test, and need to count significant predictors per block and quantify reduction
  vs univariate models.
when_to_use_negative:
- Input is raw mass-spectrometry spectra rather than a processed, sample-by-feature
  table — MAMSI's MB-PLS/MB-VIP operates on feature tables, not peak-picking.
- The goal is structural clustering, adduct/isotopologue linking, or annotation of
  the selected features — that is MamsiStructSearch territory (rt_win=5, ppm tolerance
  15 ppm), not feature counting.
- Only a single assay is available — the reduction comparison is specific to multi-assay
  MB-PLS integration vs univariate per-assay models.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: mamsi
  role: Provides MamsiPls.mb_vip_permtest for per-feature empirical p-values by permuting
    the Y outcome variable, and mb_vip for predictor importance
  repo: https://github.com/kopeckylukas/py-mamsi
- name: numpy
  role: np.where mask pattern to select feature indices below the p-value threshold
    (np.where(p_vals < 0.01))
  repo: https://github.com/kopeckylukas/py-mamsi
- name: pandas
  role: Reads the assay CSVs, adds HPOS_/LPOS_/LNEG_ prefixes, concatenates blocks,
    and slices the selected columns for per-block counting and CSV export
  repo: https://github.com/kopeckylukas/py-mamsi
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.5c01327
    title: 'MAMSI: Integration of Multiassay Liquid Chromatography–Mass Spectrometry
      Metabolomics Data Using Multiview Machine Learning'
  source_content_hash: sha256:27c8595b2c7d14781c11d4c54bb1385049f9da7bb12b92efdc2fdbcffd0664ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kopeckylukas/py-mamsi
  tools:
  - Python
  - mamsi
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mb-pls-significant-feature-counting

## Summary

Counts statistically significant LC-MS features per assay block after MB-VIP permutation testing on an MamsiPls MB-PLS model, and compares these multivariate counts against univariate-analysis counts to compute the per-experiment predictor-reduction percentages (83%, >92%, >92%, 94%) reported in the MAMSI Alzheimer's/AddNeuroMed case study.

## When to use

Use when you have fitted a multi-block PLS model over concatenated HPOS/LPOS/LNEG feature tables, obtained per-feature empirical p-values from an MB-VIP permutation test, and need to count significant predictors per block and quantify reduction vs univariate models.

## When NOT to use

- Input is raw mass-spectrometry spectra rather than a processed, sample-by-feature table — MAMSI's MB-PLS/MB-VIP operates on feature tables, not peak-picking.
- The goal is structural clustering, adduct/isotopologue linking, or annotation of the selected features — that is MamsiStructSearch territory (rt_win=5, ppm tolerance 15 ppm), not feature counting.
- Only a single assay is available — the reduction comparison is specific to multi-assay MB-PLS integration vs univariate per-assay models.

## Inputs

- Three LC-MS assay feature tables as CSV (alz_hpos.csv, alz_lpos.csv, alz_lneg.csv) with assay-prefix added (HPOS_, LPOS_, LNEG_)
- Outcome variable y encoded numerically (e.g. Gender as 1 for female, 0 for male) from alz_metadata.csv
- Fitted MamsiPls MB-PLS model with latent variables estimated via estimate_lv
- Empirical p-value vector from mb_vip_permtest (n_permutations=10000)
- Univariate-analysis significant-feature counts per experiment for the reduction comparison

## Outputs

- Count of statistically significant features per assay block (HPOS/LPOS/LNEG) at each threshold (alpha=0.01, alpha=0.05, Bonferroni-corrected)
- MB-PLS vs univariate significant-predictor reduction percentages per experiment (83%, >92%, >92%, 94%)
- significant_features_table.csv with per-experiment significant-feature counts and reductions
- Selected feature subset (x.iloc[:, mask[0]]) usable downstream for structural clustering

## How to apply

Load the three LC-MS assay feature tables (alz_hpos.csv, alz_lpos.csv, alz_lneg.csv) with column prefixes HPOS_/LPOS_/LNEG_, concatenate them column-wise, and run MamsiPls.mb_vip_permtest with n_permutations=10000 to get per-feature empirical p-values. Select significant features with the numpy mask pattern np.where(p_vals < 0.01) at alpha = 0.01, then repeat at alpha = 0.05 and at a Bonferroni-corrected threshold (alpha divided by the number of features tested per experiment), since the paper notes multiple testing correction can be applied. Map each selected feature index back to its assay block via the HPOS_/LPOS_/LNEG_ column prefixes to count significant predictors within each block. Compare the MB-PLS significant-feature counts against the univariate-analysis counts per experiment and compute the reduction; the expected result is 83% for experiment 1, >92% for experiments 2 and 3, and 94% for experiment 4.

## Related tools

- **mamsi** (Provides MamsiPls.mb_vip_permtest for per-feature empirical p-values by permuting the Y outcome variable, and mb_vip for predictor importance) — https://github.com/kopeckylukas/py-mamsi
- **numpy** (np.where mask pattern to select feature indices below the p-value threshold (np.where(p_vals < 0.01))) — https://github.com/kopeckylukas/py-mamsi
- **pandas** (Reads the assay CSVs, adds HPOS_/LPOS_/LNEG_ prefixes, concatenates blocks, and slices the selected columns for per-block counting and CSV export) — https://github.com/kopeckylukas/py-mamsi

## Examples

```
p_vals, null_vip = mamsipls.mb_vip_permtest([hpos, lpos, lneg], y, n_permutations=10000, return_scores=True); mask = np.where(p_vals < 0.01); selected = pd.concat([hpos, lpos, lneg], axis=1).iloc[:, mask[0]]
```

## Evaluation signals

- Per-experiment MB-PLS vs univariate reduction percentages match Table 2 of the paper: 83% (experiment 1), >92% (experiments 2 and 3), 94% (experiment 4).
- Significant-feature selection at alpha=0.01 reproduces exactly via np.where(p_vals < 0.01); re-running with n_permutations=10000 yields stable p-values.
- Every selected feature index maps unambiguously to one of the three blocks via its HPOS_/LPOS_/LNEG_ column prefix, and per-block counts sum to the total selected count.
- Bonferroni-corrected counts (alpha divided by the number of features tested per experiment) are <= alpha=0.01 counts for the same experiment.
- Output significant_features_table.csv contains one row per experiment with counts and reduction values consistent across all three thresholds.

## Limitations

- Reduction percentages depend on the permutation-test settings (n_permutations=10000) and threshold choice (0.01, 0.05, or Bonferroni-corrected); the paper notes multiple testing correction can be applied to adjust the p-value threshold, so counts vary with correction.
- The workflow was tested on metabolomics phenotyping data; the framework 'should be usable with other types of LC-MS data' but counts are not guaranteed to transfer.
- MB-VIP permutation testing is computationally heavy — the tutorials repository ships sample_code/permtest_hpc/ as a guide to run permutation testing on a computer cluster (Imperial HPC configuration).

## Evidence

- [methods] Permutation testing to obtain empirical p-values per feature: "p_vals, null_vip = mamsipls.mb_vip_permtest([hpos, lpos, lneg], y, n_permutations=10000, return_scores=True)"
- [methods] Significant-feature selection at p-value threshold: "mask = np.where(p_vals < 0.01)
selected = x.iloc[:, mask[0]]"
- [readme] Feature counting workflow grounded in sample data handling: "hpos = pd.read_csv('./sample_data/alz_hpos.csv').add_prefix('HPOS_')"
- [readme] Skill scope as stated by the package: "Estimation of statistically significant features (variables) using MB-VIP and permutation testing."
- [readme] Context for why block counting matters (dataset origin): [evidence span withheld — anchor status: external_doc]
