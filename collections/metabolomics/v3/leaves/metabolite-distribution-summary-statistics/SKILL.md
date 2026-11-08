---
name: metabolite-distribution-summary-statistics
description: Use when you need baseline (pre-QC) sample- and metabolite-level descriptive
  statistics for an NMR metabolomics matrix (e.g. Nightingale Health NMR data) to
  characterize distributions and decide filtering thresholds.
when_to_use_negative:
- The goal is to actually exclude samples/features — quality_control() with user-defined
  thresholds performs the filtering; this skill only characterizes the pre-filtering
  'input' layer
- The input is not a metabolomics feature matrix (e.g. already-aggregated summary
  tables or non-omics omics data) — the summarise functions expect sample x metabolite
  data ingestible by metaboprep's readers
- Only the post-QC 'qc' layer statistics are needed rather than baseline distributions
  of the raw data
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: metaboprep
  role: R package providing the Metaboprep object, read_nightingale()/read_metabolon()
    import, and sample_summary()/feature_summary() statistics on the 'input' layer
  repo: https://github.com/MRCIEU/metaboprep
- name: R
  role: Runtime environment in which the metaboprep pipeline is executed and tab-delimited
    summaries are exported
  repo: https://github.com/MRCIEU/metaboprep
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:14b73de6c47f8e1aaae624c68e523541af03186457bcc74b08fda18a032489d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MRCIEU/metaboprep
  tools:
  - R
  - metaboprep
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# metabolite-distribution-summary-statistics

## Summary

Compute per-sample and per-metabolite distribution summary statistics with the metaboprep R package on the pre-filtering 'input' layer, quantifying missingness, total peak area, and metabolite shape metrics (mean, SD, skew, coefficient of variance, Shapiro-Wilk W, outlier counts) before any quality filtering is applied.

## When to use

Use when you need baseline (pre-QC) sample- and metabolite-level descriptive statistics for an NMR metabolomics matrix (e.g. Nightingale Health NMR data) to characterize distributions and decide filtering thresholds.

## When NOT to use

- The goal is to actually exclude samples/features — quality_control() with user-defined thresholds performs the filtering; this skill only characterizes the pre-filtering 'input' layer
- The input is not a metabolomics feature matrix (e.g. already-aggregated summary tables or non-omics omics data) — the summarise functions expect sample x metabolite data ingestible by metaboprep's readers
- Only the post-QC 'qc' layer statistics are needed rather than baseline distributions of the raw data

## Inputs

- Nightingale Health NMR metabolomics matrix for ALSPAC_F24 (via read_nightingale, return_Metaboprep = TRUE)
- Metabon Excel worksheet (metabolon_v1.1_example.xlsx style, sheet e.g. 'OrigScale') via read_metabolon
- Metaboprep object with an 'input' data layer

## Outputs

- Sample-level summary data frame (missingness, total peak area, outlier occurrence count, top-10 principal components)
- Metabolite-level summary data frame (missingness, n, mean, SD, skew, coefficient of variance, Shapiro-Wilk W statistic, outlier count)
- Tab-delimited sample- and metabolite-level summary statistic files

## How to apply

Import the metabolomics matrix into a Metaboprep object (e.g. metaboprep's read_nightingale() with return_Metaboprep = TRUE for Nightingale Health NMR data, or read_metabolon() for Metabolon Excel worksheets). Run the summarise step on the 'input' source_layer with outlier_udist = 5 and tree_cut_height = 0.5 so that sample_summary() (missingness, total peak area, outlier occurrence count, top-10 principal components) and feature_summary() (missingness, n, mean, SD, skew, coefficient of variance, Shapiro-Wilk W statistic, outlier count) are computed together on raw data. The rationale for summarising the 'input' layer rather than the 'qc' layer is to describe the distributions that user-defined thresholds — sample_missingness = 0.2, feature_missingness = 0.2, total_peak_area_sd = 5, pc_outlier_sd = 5 — will act on. Export the two returned data frames as tab-delimited summary statistic files (sample-level and metabolite-level), consistent with the package goal of providing summary data as tab-delimited text files. Judge correctness by checking that both summary tables cover all input samples/features with the expected metric columns and that downstream quality_control() can reproduce exclusions (e.g. the ALSPAC_F24 Nightingale dataset excluded 3 samples and 0 metabolites under the Table 2 thresholds, versus 15 samples and 24% of metabolites for BiB_MS-1).

## Related tools

- **metaboprep** (R package providing the Metaboprep object, read_nightingale()/read_metabolon() import, and sample_summary()/feature_summary() statistics on the 'input' layer) — https://github.com/MRCIEU/metaboprep
- **R** (Runtime environment in which the metaboprep pipeline is executed and tab-delimited summaries are exported) — https://github.com/MRCIEU/metaboprep

## Examples

```
mydata <- summarise(mydata, source_layer = "input", outlier_udist = 5, tree_cut_height = 0.5)
```

## Evaluation signals

- Both summary tables contain the expected columns: sample table has missingness, total peak area, outlier count and top-10 PCs; metabolite table has missingness, n, mean, SD, skew, coefficient of variance, Shapiro-Wilk W, and outlier count
- Summaries are computed on the 'input' layer, so row/column counts equal the unfiltered dataset (no exclusions applied yet)
- Outputs are written as tab-delimited text files at both sample and metabolite level
- Downstream quality_control() with Table 2 thresholds (sample_missingness = 0.2, feature_missingness = 0.2, total_peak_area_sd = 5, outlier_udist = 5, winsorize_quantile = 1.0, tree_cut_height = 0.5, pc_outlier_sd = 5) reproduces the reference result: 3 samples and 0 metabolites excluded for ALSPAC_F24
- Cross-dataset comparison is consistent: BiB_MS-1 excludes 15 samples and 24% of metabolites under the same thresholds

## Limitations

- Package lifecycle is experimental (per README badge); APIs such as the combined summarise() call may change across versions
- PCA outlier assessment is bounded by the number of informative PCs — the README logs a warning when the requested max_num_pcs (10) exceeds available informative PCs
- metaboprep targets (un)targeted metabolite data from preformed worksheets with supported readers; unsupported platforms require the generic Metaboprep() constructor with manually supplied data/features/samples

## Evidence

- [abstract] summary statistics on both layers: "provides summary statistics and enables the user to select samples and metabolites for their analysis based on a set of quality metrics"
- [intro] tab-delimited summary outputs: "Provide useful summary data in the form of tab-delimited text file and a html report."
- [readme] raw-data summaries computed inside the pipeline: [evidence span withheld — anchor status: external_doc]
