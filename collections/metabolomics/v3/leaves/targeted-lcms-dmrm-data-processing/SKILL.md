---
name: targeted-lcms-dmrm-data-processing
description: Use when you have peak-integrated targeted LC-MS data (dMRM areas for
  metabolites and stable isotope labeled IS) in a tab-delimited file with columns
  aliquot, compound, area, type, injection_time, batch, and you need batch correction,
  outlier detection, or compound filtering.
when_to_use_negative:
- You need raw peak deconvolution, baseline noise filtering, or retention time alignment
  from vendor/centroided data - mzQuality processes peak-integrated data and does
  not replace tools like OpenMS or MZmine.
- Your data are untargeted metabolomics or proteomics without designated study QC
  (SQC), pooled QC, and blank sample types plus stable isotope labeled internal standards,
  since the batch correction and filtering metrics are defined from these sample types.
- You only need compound quality metrics without automatic correction - msQuality
  provides quality metrics but does not automatically correct the identified issues.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzQuality
  role: Primary R package used to read tab-delimited peak-integration data, build
    a SummarizedExperiment, compute compound/IS area ratios, perform SQC-based batch
    correction, flag outliers and unreliable compounds, plot QC diagnostics, and export
    reports.
  repo: https://github.com/hankemeierlab/mzQuality
- name: mzQualityDashboard
  role: Interactive Shiny dashboard built on the mzQuality package, recommended for
    interactive use and for new R users.
  repo: hankemeierlab/mzQualityDashboard
- name: R
  role: Runtime environment; mzQuality is distributed as an R package requiring version
    4.0 or later.
- name: SCIEX OS 2.1.6.59781
  role: Vendor software used to integrate the dMRM peak areas upstream of mzQuality
    processing.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:9dc92f7afd4db5600a9d1fa3fdceedb2da2228c03f80e28bcba9559eb6330418
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - SCIEX OS 2.1.6.59781
  - mzQuality
  - mzQualityDashboard
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.85
n_claims: 20
---

# targeted-lcms-dmrm-data-processing

## Summary

Quality-control and batch-correct targeted LC-MS dMRM peak-area data using the mzQuality R package, which computes compound-to-internal-standard area ratios, corrects between-batch drift from pooled study QC (SQC) samples, and flags unreliable compounds and samples.

## When to use

Use when you have peak-integrated targeted LC-MS data (dMRM areas for metabolites and stable isotope labeled IS) in a tab-delimited file with columns aliquot, compound, area, type, injection_time, batch, and you need batch correction, outlier detection, or compound filtering.

## When NOT to use

- You need raw peak deconvolution, baseline noise filtering, or retention time alignment from vendor/centroided data - mzQuality processes peak-integrated data and does not replace tools like OpenMS or MZmine.
- Your data are untargeted metabolomics or proteomics without designated study QC (SQC), pooled QC, and blank sample types plus stable isotope labeled internal standards, since the batch correction and filtering metrics are defined from these sample types.
- You only need compound quality metrics without automatic correction - msQuality provides quality metrics but does not automatically correct the identified issues.

## Inputs

- Tab-delimited peak-integration file with mandatory columns: aliquot, compound, area, type, injection_time, batch (dMRM peak areas for metabolites and stable isotope labeled internal standards)
- SummarizedExperiment object (alternative input format)
- Optional: known concentrations with calibration lines for concentration calculations

## Outputs

- SummarizedExperiment with assays ratio, ratio_corrected, and analysis results in rowData/colData
- Batch-corrected compound-to-IS area ratios (e.g. ratio_corrected.tsv export)
- HTML reports and tab-delimited colData/rowData/assay exports plus a combined Excel file (createReports)
- Per-compound quality classification: High Confidence (RSDqc <15%), With Caution (15-30%), Low Signal to Noise (RSDqc >30% or background >40%)

## How to apply

Read the tab-delimited data with mzQuality's readData(), which validates the mandatory columns (aliquot, compound, area, type, injection_time, batch), then build the SummarizedExperiment core object with buildExperiment(). Run doAnalysis() to compute compound-to-IS area ratios, apply the between-batch correction derived from pooled study QC (SQC) samples in each batch (the median ratio of every compound in all QC samples per batch yields compound-specific correction factors, per eqs 2 and 3), and calculate background signal versus study samples, matrix effect, QC-sample ratio, compound presence, and median area. Inspect batch-corrected ratios with pcaPlot(exp, assay = "ratio_corrected") and compoundPlot(exp, assay = "ratio"); deviations at batch boundaries (e.g. last two samples of batch 4 and first three of batch 5 in the example) should be removed after correction. Filter compounds using RSDqc < 30% (per metabolomics QA/QC consortium guidelines) and background e-score ≤ 40%, reading the `use` column in rowData/colData and subsetting with exp[rowData(exp)$use, exp$use]; flag Rosner-test sample outliers are excluded automatically. Finally export corrected ratios (e.g. ratio_corrected.tsv) and reports via createReports().

## Related tools

- **mzQuality** (Primary R package used to read tab-delimited peak-integration data, build a SummarizedExperiment, compute compound/IS area ratios, perform SQC-based batch correction, flag outliers and unreliable compounds, plot QC diagnostics, and export reports.) — https://github.com/hankemeierlab/mzQuality
- **mzQualityDashboard** (Interactive Shiny dashboard built on the mzQuality package, recommended for interactive use and for new R users.) — hankemeierlab/mzQualityDashboard
- **R** (Runtime environment; mzQuality is distributed as an R package requiring version 4.0 or later.)
- **SCIEX OS 2.1.6.59781** (Vendor software used to integrate the dMRM peak areas upstream of mzQuality processing.)

## Examples

```
path <- system.file("extdata", "example.tsv", package = "mzQuality"); exp <- doAnalysis(buildExperiment(readData(path)))
```

## Evaluation signals

- After doAnalysis(), the SQC samples from all batches should be aligned and mostly centered with overlapping confidence-interval ellipses in the PCA plot of ratio_corrected.
- Deviations at the batch boundary (last samples of batch 4 / first samples of batch 5 in the example) present in the pre-correction median-ratio trace should disappear after between-batch correction.
- All mandatory columns (aliquot, compound, area, type, injection_time, batch) must be present or readData() reports missing/incorrect columns.
- The `use` column in rowData/colData should be TRUE only for compounds with RSDqc ≤30% and background e-score ≤40%, and for samples that pass the Rosner test.
- NA values in the input are treated as absent data points and excluded from calculations; compounds absent in LQC/SQC are not reported.

## Limitations

- Any NA value is treated as an absent data point and will not be used for the calculations; samples with missing values will not have a data point shown in individual compound plots.
- When a compound is not detected in LQC or SQC samples, it is assumed not present and will not be reported by mzQuality.
- Calling doAnalysis() again overwrites previous assay, rowData, and colData values.
- mzQuality is a post-integration QC tool; it does not perform peak integration itself and was demonstrated on 419 EDTA plasma study samples across six batches from the CoSTREAM consortium.
- Compounds with a low signal-to-noise ratio are flagged for removal and require manual user review; automatic filtering can be overridden manually via the `use` column.

## Evidence

- [results] Batch correction derived from SQC medians: [evidence span withheld — anchor status: not_found]
- [readme] Batch correction step in doAnalysis: [evidence span withheld — anchor status: external_doc]
- [readme] readData and buildExperiment usage: [evidence span withheld — anchor status: not_found]
- [results] PCA inspection of correction outcome: [evidence span withheld — anchor status: not_found]
- [intro] Thresholds for filtering: "We defined the threshold at a relative standard deviation of QC > 30%, which follows the guidelines of the metabolomics quality assurance"
- [readme] Manual override of use column: "These values can be set manually to `TRUE` or `FALSE` to
override the automatic selection."
- [results] Outliers excluded automatically: "Potential outliers are flagged by mzQuality and are automatically excluded from further calculations"
