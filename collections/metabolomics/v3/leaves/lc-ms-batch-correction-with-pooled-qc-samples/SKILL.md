---
name: lc-ms-batch-correction-with-pooled-qc-samples
description: Use when targeted dMRM LC-MS metabolomics data spanning multiple batches
  (e.g. a 419-sample, six-batch CoSTREAM plasma run) show batch-boundary drift in
  compound/IS ratios that must be corrected before statistical analysis.
when_to_use_negative:
- Data is untargeted/open-source peak-processing output (e.g. mzML feature tables
  from OpenMS or MZmine) rather than vendor-integrated dMRM areas with compound/IS
  pairs — mzQuality processes peak-integrated data, not peaks.
- Only one batch is present or no pooled study QC (SQC) samples are distributed across
  batches — there is no SQC basis to derive between-batch correction factors.
- The goal is correcting variation that is biological rather than technical; mzQuality's
  IS-based and SQC-based adjustments target technical sources (e.g. sample 209's low
  median area was correctable via IS, indicating a technical cause).
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: mzQuality
  role: 'Performs the SQC-based between-batch correction: estimates compound-specific
    correction factors from the median ratio in QC samples per batch and multiplies
    them onto compound/IS area ratios via doAnalysis()'
  repo: https://github.com/hankemeierlab/mzQuality
- name: mzQualityDashboard
  role: Interactive R-Shiny front-end recommended for running mzQuality's batch-correction
    workflow and inspecting corrected-ratio PCA plots
  repo: https://github.com/hankemeierlab/mzQualityDashboard
- name: SCIEX OS 2.1.6.59781
  role: Upstream peak integration of the targeted dMRM LC-MS data that supplies the
    area values fed into mzQuality
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:3af90f115907e84ef97bb1109e16dc5275cde83eed3b0f671ff5c06543db9d19
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - SCIEX OS 2.1.6.59781
  - mzQuality
  - mzQualityDashboard
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# lc-ms-batch-correction-with-pooled-qc-samples

## Summary

Corrects between-batch drift in targeted LC-MS metabolomics by estimating compound-specific correction factors from pooled study quality control (SQC) samples and applying them to compound/IS area ratios, as implemented in mzQuality.

## When to use

Use when targeted dMRM LC-MS metabolomics data spanning multiple batches (e.g. a 419-sample, six-batch CoSTREAM plasma run) show batch-boundary drift in compound/IS ratios that must be corrected before statistical analysis.

## When NOT to use

- Data is untargeted/open-source peak-processing output (e.g. mzML feature tables from OpenMS or MZmine) rather than vendor-integrated dMRM areas with compound/IS pairs — mzQuality processes peak-integrated data, not peaks.
- Only one batch is present or no pooled study QC (SQC) samples are distributed across batches — there is no SQC basis to derive between-batch correction factors.
- The goal is correcting variation that is biological rather than technical; mzQuality's IS-based and SQC-based adjustments target technical sources (e.g. sample 209's low median area was correctable via IS, indicating a technical cause).

## Inputs

- Tab-delimited text file of dMRM peak-integrated areas with mandatory columns (aliquot, compound, area, type, injection_time, batch), including pooled study QC (SQC) samples spread across batches
- SummarizedExperiment object built with buildExperiment()
- Stable isotope labeled internal standard assignments per compound

## Outputs

- Batch-corrected compound/IS area ratio assay ('ratio_corrected') stored in the SummarizedExperiment
- ratio_corrected.tsv tab-delimited export
- PCA plot of batch-corrected ratios per aliquot type and batch for correction validation
- HTML/Excel report from createReports() including corrected assays and colData/rowData tables

## How to apply

Load the tab-delimited peak-integrated data (mandatory columns: aliquot, compound, area, type, injection_time, batch) with readData() and build the SummarizedExperiment via buildExperiment(); NA values are treated as absent and excluded. First compute the ratio between each compound and its assigned stable isotope labeled internal standard per injection, since correction acts on these area ratios, not raw areas. Estimate between-batch drift from the pooled study QC (SQC) samples present in each batch: the median ratio of every compound across all QC samples per batch is used to derive compound-specific correction factors, which are then multiplied by each compound's area ratios (eqs 2 and 3). Apply this via the doAnalysis() wrapper, which performs ratio calculation, SQC-based batch correction, background-signal and matrix-effect estimation, QC ratio and presence calculation, and IS suggestion; calling doAnalysis() again overwrites prior values. Validate by inspecting pcaPlot(exp, assay = "ratio_corrected") — SQC samples from all six batches should be aligned and mostly centered with overlapping confidence-interval ellipses — and by checking that batch-boundary deviations (e.g. the last two samples of batch 4 and first three of batch 5 in the example) are removed in the median-ratio trace. Export the corrected assay as ratio_corrected.tsv via createReports().

## Related tools

- **mzQuality** (Performs the SQC-based between-batch correction: estimates compound-specific correction factors from the median ratio in QC samples per batch and multiplies them onto compound/IS area ratios via doAnalysis()) — https://github.com/hankemeierlab/mzQuality
- **mzQualityDashboard** (Interactive R-Shiny front-end recommended for running mzQuality's batch-correction workflow and inspecting corrected-ratio PCA plots) — https://github.com/hankemeierlab/mzQualityDashboard
- **SCIEX OS 2.1.6.59781** (Upstream peak integration of the targeted dMRM LC-MS data that supplies the area values fed into mzQuality)

## Examples

```
path <- system.file("extdata", "example.tsv", package = "mzQuality"); exp <- buildExperiment(readData(path)); exp <- doAnalysis(exp = exp)
```

## Evaluation signals

- PCA of the corrected ratio assay shows SQC samples from all six batches aligned and mostly centered, with overlapping confidence-interval ellipses across batches.
- Median-ratio trace across the batch boundary no longer shows deviations at the last two samples of batch 4 and first three samples of batch 5 (example dataset).
- The corrected assay is written as ratio_corrected.tsv and the SummarizedExperiment contains an assay named 'ratio_corrected' after doAnalysis().
- SQC-derived median ratios per batch should be centered around the same level across batches; NA values were excluded from calculations, so corrected values are absent where input was NA.
- Re-running doAnalysis() reproduces or overwrites the same corrected values rather than accumulating drift.

## Limitations

- NA values are treated as absent data points and excluded from calculations; compounds not detected in LQC or SQC samples are assumed not present and not reported.
- Between-batch correction relies entirely on SQC samples being present and representative in every batch; corrections act on compound/IS area ratios, not raw areas, so IS quality matters.
- The example dataset is not identified by name or accession, and no changelog or specific mzQuality version is documented, so reproducibility relies on the GitHub snapshot used.
- Deviations correctable via IS (e.g. sample 209) suggest some variation is handled by IS correction rather than batch correction; distinguishing technical from biological sources requires inspection.

## Evidence

- [other] SQC-based batch correction: median ratio of every compound in all QC samples per batch drives correction factors (eqs 2 and 3): [evidence span withheld — anchor status: not_found]
- [readme] doAnalysis() wrapper performs ratio calculation and SQC-based batch correction among other metrics: "1.  Calculate the ratio between the compounds and assigned internal standards,
2.  Perform batch correction using the pooled study quality control samples (SQC),"
- [results] PCA validation of corrected ratios shows SQC samples aligned and centered across batches: [evidence span withheld — anchor status: not_found]
- [results] Batch-boundary deviations in median ratio are removed by the between-batch adjustment: [evidence span withheld — anchor status: not_found]
- [results] NA handling rule during analysis: "Any NA value is treated as an absent data point and will not be used for the calculations"
- [methods] readData checks mandatory columns on the tab-delimited input: "you can use the `readData` function. This function will read in your data and check for any missing or incorrect columns."
- [readme] Example invocation grounded in README: build experiment and run analysis: "path <- system.file("extdata", "example.tsv", package = "mzQuality")
exp <- buildExperiment(readData(path))"
