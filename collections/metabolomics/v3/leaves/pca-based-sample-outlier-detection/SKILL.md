---
name: pca-based-sample-outlier-detection
description: Use when a raw metabolite-by-sample matrix (e.g. Nightingale NMR or Metabolon
  UPLC-MS/MS export) is being QC-filtered in metaboprep and samples with abnormal
  multivariate metabolite profiles must be flagged via PCA scores at a user-defined
  SD cutoff.
when_to_use_negative:
- The dataset is already a curated feature table with no sample-level QC pending —
  PCA outlier flagging is part of metaboprep's preanalysis filtering, not downstream
  analysis
- Only univariate outlier screening is required (outlier_udist on individual metabolites)
  rather than multivariate PC-score screening
- The input is not a sample x metabolite platform export that read_metabolon/read_nightingale
  can parse into a Metaboprep object
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_3172
tools:
- name: metaboprep
  role: Implements the quality_control() pipeline whose PCA step scores samples on
    the top PCs and excludes those beyond pc_outlier_sd standard deviations
  repo: https://github.com/MRCIEU/metaboprep
- name: R
  role: Runtime environment in which the metaboprep quality_control() and summary()
    calls are executed
  repo: https://github.com/MRCIEU/metaboprep
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:bb361f0084460ae95ac3d6969366832e50d545f3a7f460983c1eaa9fc401631a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MRCIEU/metaboprep
  tools:
  - R
  - metaboprep
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# PCA-based sample outlier detection

## Summary

A metabolomics quality-control step in the metaboprep R package that runs principal component analysis on the sample summary layer and flags samples whose PC scores lie beyond a user-defined standard-deviation cutoff (pc_outlier_sd, e.g. 5), adding them to the QC exclusion set.

## When to use

Use when a raw metabolite-by-sample matrix (e.g. Nightingale NMR or Metabolon UPLC-MS/MS export) is being QC-filtered in metaboprep and samples with abnormal multivariate metabolite profiles must be flagged via PCA scores at a user-defined SD cutoff.

## When NOT to use

- The dataset is already a curated feature table with no sample-level QC pending — PCA outlier flagging is part of metaboprep's preanalysis filtering, not downstream analysis
- Only univariate outlier screening is required (outlier_udist on individual metabolites) rather than multivariate PC-score screening
- The input is not a sample x metabolite platform export that read_metabolon/read_nightingale can parse into a Metaboprep object

## Inputs

- Metaboprep object containing an 'input' data layer built from a platform export (e.g. Metabolon OrigScale xlsx sheet, Nightingale NMR xlsx) via read_metabolon()/read_nightingale(return_Metaboprep=TRUE)
- User-defined QC thresholds, notably pc_outlier_sd (e.g. 5) plus tree_cut_height (e.g. 0.5) for feature-independence determination

## Outputs

- QC data layer with samples flagged/excluded under the user_defined_sample_pca_outlier exclusion code
- summary() exclusion-count table listing user_defined_sample_pca_outlier counts
- Tab-delimited sample summary including top-10 principal component scores and outlier occurrence counts

## How to apply

Import the platform export into a Metaboprep object (read_metabolon or read_nightingale with return_Metaboprep = TRUE), then call quality_control() on source_layer = 'input' with pc_outlier_sd = 5 together with the companion thresholds sample_missingness = 0.2, feature_missingness = 0.2, total_peak_area_sd = 5, outlier_udist = 5, winsorize_quantile = 1.0 and tree_cut_height = 0.5. The pipeline first computes sample summary statistics (missingness, total peak area, top-10 PCs), copies the input data to a new 'qc' layer, applies the missingness and total-peak-area screens, and then runs PCA outlier assessment on the informative PCs, excluding samples whose scores exceed +/- pc_outlier_sd standard deviations. Feature independence for the PCA is re-identified during this step, informed by the feature dendrogram cut at tree_cut_height. Excluded samples receive the exclusion code user_defined_sample_pca_outlier, which is tallied by summary() on the Metaboprep object.

## Related tools

- **metaboprep** (Implements the quality_control() pipeline whose PCA step scores samples on the top PCs and excludes those beyond pc_outlier_sd standard deviations) — https://github.com/MRCIEU/metaboprep
- **R** (Runtime environment in which the metaboprep quality_control() and summary() calls are executed) — https://github.com/MRCIEU/metaboprep

## Examples

```
mydata <- mydata |> quality_control(source_layer="input", sample_missingness=0.2, feature_missingness=0.2, total_peak_area_sd=5, outlier_udist=5, outlier_treatment="leave_be", winsorize_quantile=1.0, tree_cut_height=0.5, pc_outlier_sd=5)
```

## Evaluation signals

- summary() on the quality-controlled Metaboprep object reports a user_defined_sample_pca_outlier count consistent with the chosen pc_outlier_sd threshold (e.g. 0 on the metabolon_v1.1 example at +/- 5 Sdev)
- The console log shows the step 'Running sample data PCA outlier analysis at +/- 5 Sdev' followed by completion without exclusion of expected-retained samples
- Reproduced exclusion counts match published cohort results (BiB_MS-1: 15 samples; ALSPAC_F24: 3 samples, 0 metabolites) when Table 2 thresholds are used
- The returned sample summary data frame contains PC columns (top-10 PCs) and the outlier occurrence count needed for the PCA screen
- The 'qc' layer differs from the 'input' layer only by samples carrying exclusion reasons, not by altered metabolite values (outlier_treatment = 'leave_be')

## Limitations

- The stated maximum number of PCs (max_num_pcs = 10) can exceed the number of available informative PCs, triggering a warning and reduced PCA outlier assessment (seen on the example dataset with only 2 informative PCs)
- The package is marked lifecycle: experimental, so pipeline behaviour and exclusion codes may change between versions
- With winsorize_quantile = 1.0 no winsorization is applied, so PCA scores are computed on unmodified values and extreme samples can drive PC axes

## Evidence

- [intro] intro quote on standard filtering pipeline: "Perform data filtering on the data set using a standard pipeline and according to user-defined thresholds."
- [abstract] abstract quote on quality-metric-based selection: "enables the user to select samples and metabolites for their analysis based on a set of quality metrics"
- [readme] readme console log of the PCA outlier step: [evidence span withheld — anchor status: external_doc]
- [readme] readme console log on feature independence re-identification during PCA outlier analysis: [evidence span withheld — anchor status: external_doc]
- [readme] readme exclusion code tally in object summary: [evidence span withheld — anchor status: external_doc]
