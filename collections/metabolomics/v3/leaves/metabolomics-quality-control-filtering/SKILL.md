---
name: metabolomics-quality-control-filtering
description: Use when raw platform-exported metabolomics data (e.g. Metabolon OrigScale
  xlsx, Nightingale NMR, Olink, SomaLogic adat) must be QC'd, filtered on missingness/outlier
  thresholds, and summarized before association analysis.
when_to_use_negative:
- Input is already a QC-filtered feature table with no sample metadata — the package
  expects preformed worksheets with features/samples annotations
- Analysis requires statistical testing or association modeling; metaboprep is explicitly
  a preanalysis data description and processing tool
- Non-metabolomics omics data outside the supported platform formats (Metabolon, Nightingale,
  Olink, SomaLogic, dummy CSV)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: metaboprep
  role: Core R package providing read_metabolon, Metaboprep object constructor, quality_control
    pipeline, summaries, and report generation
  repo: https://github.com/MRCIEU/metaboprep
- name: readxl
  role: Reads the Metabolon xlsx example workbook during data import
- name: rmarkdown
  role: Renders the qc_report HTML quality summary
- name: dendextend
  role: Supports plotting the feature-clustering dendrogram of the QC tree
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_007
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:8edd54a8ebd96fc72df83f379cf7cca7229fe71fc831e970e76de301b9eba07e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MRCIEU/metaboprep
  tools:
  - R
  - dendextend
  - knitr
  - metaboprep
  - readxl
  - rmarkdown
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# metabolomics-quality-control-filtering

## Summary

Standardized preanalysis quality-control and filtering of (un)targeted metabolomics datasets using the metaboprep R package, which reads preformed worksheets, computes sample/feature quality summaries, excludes samples and metabolites against user-defined thresholds, and emits tab-delimited summaries plus an HTML report.

## When to use

Use when raw platform-exported metabolomics data (e.g. Metabolon OrigScale xlsx, Nightingale NMR, Olink, SomaLogic adat) must be QC'd, filtered on missingness/outlier thresholds, and summarized before association analysis.

## When NOT to use

- Input is already a QC-filtered feature table with no sample metadata — the package expects preformed worksheets with features/samples annotations
- Analysis requires statistical testing or association modeling; metaboprep is explicitly a preanalysis data description and processing tool
- Non-metabolomics omics data outside the supported platform formats (Metabolon, Nightingale, Olink, SomaLogic, dummy CSV)

## Inputs

- Metabolon-formatted xlsx metabolite matrix (sheet 'OrigScale', e.g. metabolon_v1.1_example.xlsx)
- Nightingale NMR, Olink txt, or SomaLogic .adat platform exports supported by read_metabolon/package readers
- User-defined QC thresholds (sample_missingness, feature_missingness, total_peak_area_sd, outlier_udist, outlier_treatment, winsorize_quantile, tree_cut_height, pc_outlier_sd)

## Outputs

- Metaboprep S7 object with 'input' and 'qc' data layers
- Tab-delimited filtered dataset and QC summary files (e.g. qc_filtered_dataset.txt, qc_exclusion_summary.txt)
- HTML quality report (qc_report rmarkdown template) with exclusion summary, PC plots, dendrogram, and batch-variable eta-squared boxplots

## How to apply

Load the metabolite matrix via read_metabolon (e.g. sheet='OrigScale' for Metabolon files, return_Metaboprep=TRUE) and build a Metaboprep object from data, features, and samples slots. Run quality_control() on source_layer='input' with explicit thresholds — sample_missingness=0.2, feature_missingness=0.2, total_peak_area_sd=5, outlier_udist=5, outlier_treatment='leave_be', winsorize_quantile=1.0, tree_cut_height=0.5, pc_outlier_sd=5 — which first excludes extreme (>=80%) sample/feature missingness, then applies user-defined missingness cutoffs, total-peak-area and univariate outlier flags, and PCA-based sample outlier detection (re-identifying feature independence per PC run). Inspect the pipeline log lines and summary() exclusion codes to reconcile how many samples/metabolites were removed at each step. Produce feature_summary and sample_summary tables as tab-delimited text plus a rendered HTML qc_report so filtering decisions are openly disclosed.

## Related tools

- **metaboprep** (Core R package providing read_metabolon, Metaboprep object constructor, quality_control pipeline, summaries, and report generation) — https://github.com/MRCIEU/metaboprep
- **readxl** (Reads the Metabolon xlsx example workbook during data import)
- **rmarkdown** (Renders the qc_report HTML quality summary)
- **dendextend** (Supports plotting the feature-clustering dendrogram of the QC tree)

## Examples

```
mydata <- read_metabolon(system.file("extdata","metabolon_v1.1_example.xlsx",package="metaboprep"), sheet="OrigScale", return_Metaboprep=TRUE); mydata <- quality_control(mydata, source_layer="input", sample_missingness=0.2, feature_missingness=0.2, total_peak_area_sd=5, outlier_udist=5, outlier_treatment="leave_be", winsorize_quantile=1.0, tree_cut_height=0.5, pc_outlier_sd=5)
```

## Evaluation signals

- QC log reports the sequential exclusion steps: extreme missingness (>=80%), user-defined missingness (>=20%), total peak abundance +/- 5 Sdev, and PCA outlier analysis at +/- 5 Sdev
- summary() on the object lists expected exclusion codes (user_defined_sample_missingness, user_defined_feature_missingness, etc.) with counts consistent with the log
- Reproduced cohort counts match reported values (BiB_MS-1: 15 samples excluded, 24% of metabolites; ALSPAC_F24: 3 samples, 0 metabolites)
- Object summary shows two data layers named 'input' and 'qc' with sample/feature summary layers for both
- Rendered HTML report (e.g. myproject_metaboprep_qc_report.html) exists in output_dir and displays PC plots, dendrogram, and Shapiro W-statistic histograms

## Limitations

- Package is marked 'experimental' lifecycle in the README
- PCA outlier step warns when max_num_pcs exceeds available informative PCs (e.g. 'stated max PCs [max_num_pcs=10] ... greater than the number of available informative PCs [2]')
- The paper reports no quantitative validation metric for individual filtering sub-steps in isolation; acceptance is qualitative (e.g. each dendrogram cluster contributing one representative)

## Evidence

- [methods] standardized QC pipeline thresholds: "sample_missingness  = 0.2, feature_missingness = 0.2, total_peak_area_sd  = 5, outlier_udist       = 5, outlier_treatment   = "leave_be""
- [readme] package goal statement: "Perform data filtering on the data set using a standard pipeline and according to user-defined thresholds."
- [readme] QC pipeline execution example: "mydata <- mydata |> quality_control( source_layer        = "input", sample_missingness  = 0.2, feature_missingness = 0.2"
- [abstract] abstract scope of package: "The package extracts data from preformed worksheets, provides summary statistics and enables the user to select samples and metabolites for their analysis based on a set of quality metrics"
- [abstract] reporting purpose: "A report summarizing quality metrics and the influence of available batch variables on the data are generated for the purpose of open disclosure"
