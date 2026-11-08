---
name: internal-standard-ratio-normalization
description: Use when raw peak areas from a targeted LC-MS method with stable isotope
  labeled internal standards must be converted to compound/IS ratios before quality
  assessment, batch correction or outlier detection.
when_to_use_negative:
- The data are untargeted full-scan features with no assigned stable isotope labeled
  internal standard, so no compound/IS pairing exists
- The peak table has not yet been integrated by vendor software (e.g. SCIEX OS) and
  lacks the area and compound columns mzQuality requires
- Only raw areas are needed for reporting and no QC, batch correction or outlier assessment
  will be performed
edam_operation: http://edamontology.org/operation_3791
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzQuality
  role: Computes compound/IS area ratios via doAnalysis() and stores them in the SummarizedExperiment
    assay for all subsequent quality checks
  repo: https://github.com/hankemeierlab/mzQuality
- name: mzQualityDashboard
  role: Interactive Shiny front-end on top of the mzQuality package that exposes the
    ratio-based analysis results for visual inspection
  repo: https://github.com/hankemeierlab/mzQualityDashboard
- name: SCIEX OS
  role: Vendor software used upstream to integrate the dMRM peaks that supply the
    raw compound and IS areas fed into ratio calculation
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_006
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:195f8fa6d0392891acf8715b86da43dd4fe9bfa312df0b8d328fd2df94b0b9fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - SCIEX OS
  - mzQuality
  - mzQualityDashboard
  edam_operation: http://edamontology.org/operation_3791
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Internal-standard ratio normalization

## Summary

Normalizes targeted LC-MS dMRM peak areas by dividing each metabolite's area by its assigned stable isotope labeled internal standard (IS) to compute compound/IS area ratios, the core measurement mzQuality uses for all downstream QC, batch correction and outlier detection.

## When to use

Use when raw peak areas from a targeted LC-MS method with stable isotope labeled internal standards must be converted to compound/IS ratios before quality assessment, batch correction or outlier detection.

## When NOT to use

- The data are untargeted full-scan features with no assigned stable isotope labeled internal standard, so no compound/IS pairing exists
- The peak table has not yet been integrated by vendor software (e.g. SCIEX OS) and lacks the area and compound columns mzQuality requires
- Only raw areas are needed for reporting and no QC, batch correction or outlier assessment will be performed

## Inputs

- Tab-delimited peak table (mzQuality example.tsv format) with columns aliquot, compound, area, type, injection_time, batch, and compound_is/area_is pairing for each compound's stable isotope labeled internal standard
- SummarizedExperiment object built by mzQuality's buildExperiment()
- Optional concentration column with known spike concentrations for calibration-line samples

## Outputs

- compound/IS area ratio values stored in the 'ratio' assay of the SummarizedExperiment
- Ratios used downstream for batch-corrected 'ratio_corrected' assay, QC outlier flags and background signal (e-score) metrics
- Optionally, absolute concentrations fitted from calibration lines applied to the ratio values

## How to apply

Load the tab-delimited peak table with mzQuality's readData() and verify the mandatory columns aliquot, compound, area, type, injection_time and batch (plus concentration for calibration lines) are present, then build the SummarizedExperiment with buildExperiment(). Call doAnalysis(), whose first step is to 'Calculate the ratio between the compounds and assigned internal standards' so that every metabolite is expressed as a compound/IS ratio rather than a raw area. Ratios are stored in the 'ratio' assay of the SummarizedExperiment and feed directly into batch correction using pooled study QC (SQC) samples, Rosner-test outlier flagging, background (e-score) calculation and matrix-effect estimation; re-running doAnalysis overwrites previous assay values. Compounds with missing values (NA) have no data point, since any NA is treated as an absent data point and excluded from calculations, so ratios are only computed where both the compound and its IS were integrated.

## Related tools

- **mzQuality** (Computes compound/IS area ratios via doAnalysis() and stores them in the SummarizedExperiment assay for all subsequent quality checks) — https://github.com/hankemeierlab/mzQuality
- **mzQualityDashboard** (Interactive Shiny front-end on top of the mzQuality package that exposes the ratio-based analysis results for visual inspection) — https://github.com/hankemeierlab/mzQualityDashboard
- **SCIEX OS** (Vendor software used upstream to integrate the dMRM peaks that supply the raw compound and IS areas fed into ratio calculation)

## Examples

```
exp <- doAnalysis(buildExperiment(readData(system.file("extdata", "example.tsv", package = "mzQuality"))))
```

## Evaluation signals

- The 'ratio' assay in the SummarizedExperiment is populated for every compound/IS pair where neither value is NA, and NA pairs contribute no data point as mzQuality treats NA as absent
- Rerunning doAnalysis reproduces the same ratios (it overwrites prior assay values rather than appending)
- SQC samples across all six batches, once batch-corrected from these ratios, align with overlapping confidence-interval ellipses in the pcaPlot of ratio_corrected
- Outlier samples flagged by the Rosner test (e.g. sample 209 with a notably lower median IS area) are identifiable because their ratio deviations track the IS area anomaly
- readData() reports no missing or incorrect mandatory columns (aliquot, compound, area, type, injection_time, batch) before buildExperiment() is called

## Limitations

- Any NA value is treated as an absent data point and will not be used for the calculations, so gaps in compound or IS integration silently drop ratios
- When a compound is not detected in LQC or SQC samples, the compound will be assumed to not be present and will not be reported by mzQuality
- All calculations are added to the assay, rowData and colData slots or overwrite existing values, so calling doAnalysis again replaces previous ratio values rather than versioning them

## Evidence

- [readme] doAnalysis computes compound/IS ratios as its first step: [evidence span withheld — anchor status: external_doc]
- [readme] doAnalysis is a single wrapper covering all analyses including the ratio calculation: [evidence span withheld — anchor status: not_found]
- [readme] mzQuality input is a tab-delimited file with mandatory columns: [evidence span withheld — anchor status: external_doc]
- [results] NA values are excluded from ratio calculations: "Any NA value is treated as an absent data point and will not be used for the calculations"
- [methods] mzQuality targets peak-integrated dMRM data from a polarity-switching targeted LC-MS method: "a targeted LC-MS method is operated in polarity switching mode where all metabolites and the corresponding stable isotope labeled IS are analyzed in dynamic Multiple Reaction Monitoring (dMRM) mode"
- [other] Sample outliers are assessed from the internal standard areas rather than raw compound signal: [evidence span withheld — anchor status: not_found]
