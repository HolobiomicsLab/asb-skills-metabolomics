---
name: pc-outlier-visualization
description: Use when a Metaboprep object's QC run needs PCA-based sample outlier
  identification and PC plots for the HTML QC report — e.g. flagging samples beyond
  ±5 SD on informative PCs under pc_outlier_sd=5.
when_to_use_negative:
- Input is already a QC-filtered sample/feature table with no Metaboprep object or
  sample annotation — PCA outlier flags depend on the object's qc data layer and metadata.
- The goal is feature-level (metabolite) filtering or batch-variable eta-squared association
  analysis rather than sample-level PC outlier inspection.
- The dataset yields very few informative PCs (the pipeline warns max_num_pcs=10 exceeds
  available informative PCs, e.g. 2), making PC outlier visualization uninformative.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: metaboprep
  role: Performs the ±5 SD PCA-based sample outlier assessment inside quality_control
    and supplies PC plots and the feature tree to the qc_report HTML template
  repo: https://github.com/MRCIEU/metaboprep
- name: rmarkdown
  role: Renders the qc_report template into the HTML report in which the PC outlier
    plots are displayed
- name: R
  role: Runtime for the metaboprep package and the rmarkdown report generation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btac059
    title: 'metaboprep: an R package for preanalysis data description and processing'
  source_content_hash: sha256:deb677a0743a9ee682a4166e12ccd7476caf0a36038d497370b973e7ec4e6293
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MRCIEU/metaboprep
  tools:
  - R
  - metaboprep
  - rmarkdown
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# pc-outlier-visualization

## Summary

Identifies PCA-based sample outliers during metaboprep's quality_control run (pc_outlier_sd=5) and surfaces PC plots in the qc_report HTML, giving transparent preanalysis QC of cohorts such as BiB_MS-1 and ALSPAC_F24.

## When to use

Use when a Metaboprep object's QC run needs PCA-based sample outlier identification and PC plots for the HTML QC report — e.g. flagging samples beyond ±5 SD on informative PCs under pc_outlier_sd=5.

## When NOT to use

- Input is already a QC-filtered sample/feature table with no Metaboprep object or sample annotation — PCA outlier flags depend on the object's qc data layer and metadata.
- The goal is feature-level (metabolite) filtering or batch-variable eta-squared association analysis rather than sample-level PC outlier inspection.
- The dataset yields very few informative PCs (the pipeline warns max_num_pcs=10 exceeds available informative PCs, e.g. 2), making PC outlier visualization uninformative.

## Inputs

- Metaboprep S7 object with 'input'/'qc' data layers and sample annotation (sample_id, run_day, box_id, lot, reason_excluded, excluded), built via read_metabolon(sheet='OrigScale') + Metaboprep(data, features, samples)
- Metabolon-formatted xlsx input (e.g. metabolon_v1.1_example.xlsx)
- User-defined QC thresholds including pc_outlier_sd (study value: 5) and outlier_treatment ('leave_be')

## Outputs

- PC plots embedded in the qc_report HTML (e.g. BiB_MS-1_metaboprep_qc_report.html, ALSPAC_F24_metaboprep_qc_report.html)
- Sample-level PCA outlier flags surfaced as user_defined_sample_pca_outlier counts in the exclusion-codes summary
- sample_summary QC layer with PCA outlier metrics for the rendered report

## How to apply

Run the metaboprep quality_control pipeline on a Metaboprep object (built via read_metabolon(..., sheet='OrigScale') then Metaboprep(data, features, samples)), passing pc_outlier_sd=5 alongside the study's other thresholds (sample_missingness=0.2, feature_missingness=0.2, total_peak_area_sd=5, outlier_udist=5, outlier_treatment='leave_be', winsorize_quantile=1.0, tree_cut_height=0.5). The pipeline runs PCA on the 'qc' data layer, re-identifies feature independence and the informative PCs, and flags samples lying beyond ±5 SD on those PCs; flagged samples receive the user_defined_sample_pca_outlier exclusion code in the object's exclusion-codes summary. PC plots are then rendered into the qc_report rmarkdown template via generate_report(format='html'), as displayed in the deposited BiB_MS-1 and ALSPAC_F24 reports (Supplementary Data S2 and S3). Treat the SD cutoff as user-definable — the package explicitly gives users flexibility in selection thresholds — and note that on small datasets the pipeline warns when max_num_pcs=10 exceeds the number of available informative PCs (e.g. 2), limiting outlier assessment.

## Related tools

- **metaboprep** (Performs the ±5 SD PCA-based sample outlier assessment inside quality_control and supplies PC plots and the feature tree to the qc_report HTML template) — https://github.com/MRCIEU/metaboprep
- **rmarkdown** (Renders the qc_report template into the HTML report in which the PC outlier plots are displayed)
- **R** (Runtime for the metaboprep package and the rmarkdown report generation)

## Examples

```
mydata |> quality_control(source_layer="input", sample_missingness=0.2, feature_missingness=0.2, pc_outlier_sd=5, outlier_treatment="leave_be")
```

## Evaluation signals

- Console log shows the 'Running sample data PCA outlier analysis at +/- 5 Sdev' step completing within quality_control.
- summary(mydata) exclusion-codes table reports counts under user_defined_sample_pca_outlier consistent with the ±5 SD rule.
- Rendered qc_report HTML contains PC plots alongside the raw-data overview, exclusion summary, Shapiro W-statistic histograms and feature dendrogram (matching Supplementary Data S2/S3).
- Changing pc_outlier_sd monotonically changes the number of flagged samples (smaller SD → more exclusions), confirming the threshold drives the calls.
- On small datasets, confirm the reported informative PC count before interpreting outlier calls, since max_num_pcs=10 may exceed available informative PCs.

## Limitations

- metaboprep is marked 'experimental' lifecycle in the README, so APIs may change.
- On datasets with few informative PCs the PCA outlier step warns that max_num_pcs=10 exceeds the available informative PCs (e.g. 2), limiting the assessment.
- The study ran with outlier_treatment='leave_be' and winsorize_quantile=1.0, so PC outlier flags are reported rather than auto-removed, and flags depend entirely on the user-chosen ±5 SD cutoff.

## Evidence

- [readme] PCA outlier analysis runs at ±5 SD within quality_control: [evidence span withheld — anchor status: external_doc]
- [readme] Informative PC count can limit the PCA outlier assessment: [evidence span withheld — anchor status: external_doc]
- [abstract] HTML report discloses quality metrics and batch-variable influence: "A report summarizing quality metrics and the influence of available batch variables on the data are generated for the purpose of open disclosure"
- [intro] Filtering uses a standard pipeline with user-defined thresholds: "Perform data filtering on the data set using a standard pipeline and according to user-defined thresholds."
