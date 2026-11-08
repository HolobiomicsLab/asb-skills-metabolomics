---
name: benjamini-hochberg-fdr-correction
description: Use when a feature table from LC/MS preprocessing (apLCMS, XCMS, or two-stage
  apLCMS, batch-corrected or not) has per-feature t-test p-values between two cell
  types and you need multiplicity-adjusted significance counts at a lenient FDR cutoff
  such as 0.2.
when_to_use_negative:
- Input is raw mass-spectrometry profiles (NetCDF/mzML) that still need peak detection,
  RT correction, and feature alignment via apLCMS preprocessing — apply the preprocessing
  skill first
- The comparison involves more than two groups or a continuous outcome, where a two-sample
  t-test per feature is inappropriate
- Only a handful of features are tested, where BH adjustment across features adds
  little and a stricter per-feature threshold may be preferable
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Environment in which the per-feature two-sample t-tests, Benjamini-Hochberg
    FDR adjustment, and significant-feature counting are performed on the feature
    matrices
- name: apLCMS
  role: Source of the feature-intensity matrices (traditional and two-stage preprocessing)
    whose t-test p-values are FDR-adjusted
  repo: https://github.com/tianwei-yu/apLCMS
- name: ComBat (sva)
  role: Empirical-Bayes batch correction applied to intensities before t-testing;
    BH FDR adjustment is then run on the corrected data's p-values
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-70850-0
    title: Addressing the batch effect issue for LC/MS metabolomics data in data preprocessing
  source_content_hash: sha256:5acf35b456c4330c277446897678c3546230a87f85affd742b0558920a97508f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tianwei-yu/apLCMS
  tools:
  - ComBat (sva)
  - R
  - apLCMS
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# benjamini-hochberg-fdr-correction

## Summary

Apply Benjamini-Hochberg FDR adjustment to per-feature two-sample t-test p-values computed on LC/MS metabolomics feature-intensity matrices (e.g. ST000868, oak vs. wine yeast), then count significant features at FDR <= 0.2 to reproduce the paper's Fig. 3d/3e comparisons.

## When to use

Use when a feature table from LC/MS preprocessing (apLCMS, XCMS, or two-stage apLCMS, batch-corrected or not) has per-feature t-test p-values between two cell types and you need multiplicity-adjusted significance counts at a lenient FDR cutoff such as 0.2.

## When NOT to use

- Input is raw mass-spectrometry profiles (NetCDF/mzML) that still need peak detection, RT correction, and feature alignment via apLCMS preprocessing — apply the preprocessing skill first
- The comparison involves more than two groups or a continuous outcome, where a two-sample t-test per feature is inappropriate
- Only a handful of features are tested, where BH adjustment across features adds little and a stricter per-feature threshold may be preferable

## Inputs

- Feature-intensity matrix (samples x features) from apLCMS, XCMS, or two-stage apLCMS on the ST000868 dataset
- Per-feature two-sample t-test p-values comparing oak vs. wine yeast cell types
- Cell-type annotation labels (oak vs. wine yeast) and, for the corrected analysis, batch labels used by ComBat (sva)
- FDR cutoff threshold (0.2)

## Outputs

- Per-feature BH-adjusted p-values (FDR) within each preprocessing parameter setting
- Counts table of features with FDR <= 0.2 per parameter setting (e.g. fig3d_significant_feature_counts.csv or fig3e_fdr_significant_counts.csv)

## How to apply

For each feature-intensity matrix (per apLCMS parameter setting, with or without ComBat correction), first perform a two-sample t-test comparing oak versus wine yeast intensity values feature by feature, after filtering features to those with <= 33% zero (missing) values within at least one cell type for the uncorrected analysis. Then apply the Benjamini-Hochberg procedure across all features within each parameter setting to convert p-values to adjusted p-values (FDR). Count features with adjusted p-value <= 0.2 and write the per-setting counts table (e.g. fig3d_significant_feature_counts.csv for uncorrected data, fig3e_fdr_significant_counts.csv for ComBat-corrected data). The rationale is that thousands of features are tested simultaneously, so raw p-values would massively inflate false positives; BH adjustment within each parameter setting keeps the comparison between preprocessing methods (two-stage apLCMS vs. traditional apLCMS vs. XCMS) fair.

## Related tools

- **R** (Environment in which the per-feature two-sample t-tests, Benjamini-Hochberg FDR adjustment, and significant-feature counting are performed on the feature matrices)
- **apLCMS** (Source of the feature-intensity matrices (traditional and two-stage preprocessing) whose t-test p-values are FDR-adjusted) — https://github.com/tianwei-yu/apLCMS
- **ComBat (sva)** (Empirical-Bayes batch correction applied to intensities before t-testing; BH FDR adjustment is then run on the corrected data's p-values)

## Examples

```
p_adj <- p.adjust(p_values, method = "BH"); sig_counts <- sum(p_adj <= 0.2)
```

## Evaluation signals

- Each adjusted p-value vector has the same length and feature ordering as the input t-test p-value vector, with adjusted values monotonically >= raw p-values
- The output counts table contains one significant-feature count (FDR <= 0.2) for every parameter setting, with no missing settings
- Each reported count equals exactly the number of features whose BH-adjusted p-value is <= 0.2, verifiable by recomputing from the adjusted p-value vector
- The qualitative pattern across preprocessing methods (two-stage apLCMS detecting more significant features at FDR <= 0.2 than traditional apLCMS and XCMS) matches Fig. 3d (uncorrected) and Fig. 3e (ComBat-corrected) of the paper
- Uncorrected-data counts are relatively low overall, consistent with the paper's finding that all methods yielded relatively few significant metabolites without batch correction

## Limitations

- Post hoc batch-effect correction methods such as ComBat can only adjust signal intensity and cannot fix feature misalignment or incomplete weak signal recovery, so FDR-adjusted counts on corrected data are bounded by preprocessing quality
- The lenient FDR <= 0.2 cutoff means results are reported as relative comparisons between methods rather than confirmatory biomarker lists
- Traditional preprocessing treating all samples as a single group can cause peak alignment errors that no downstream statistical adjustment can correct

## Evidence

- [other] For every retained feature, perform a two-sample t-test comparing oak versus wine yeast intensity values.: [evidence span withheld — anchor status: not_found]
- [other] BH correction step in the uncorrected workflow: [evidence span withheld — anchor status: not_found]
- [other] Threshold and counting step: [evidence span withheld — anchor status: not_found]
- [other] ComBat-corrected analysis workflow: [evidence span withheld — anchor status: not_found]
- [other] Rationale for lenient threshold finding: [evidence span withheld — anchor status: not_found]
