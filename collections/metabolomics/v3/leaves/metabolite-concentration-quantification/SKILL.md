---
name: metabolite-concentration-quantification
description: Use when targeted LC-MS/MS data include calibration-line samples with
  known spike concentrations in a 'concentration' column and you need absolute concentrations
  (not just ratios) for study samples.
when_to_use_negative:
- The dataset has no calibration-line samples with known concentrations — mzQuality
  then only computes ratios, batch correction and QC metrics, not absolute concentrations.
- More than one calibration sample type is present — the linear fit is restricted
  to a single calibration sample type, so multi-type calibration is out of scope.
- Peak integration has not been performed upstream (mzQuality processes peak-integrated
  data, not raw vendor files).
edam_operation: http://edamontology.org/operation_3215
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzQuality
  role: Provides readData/buildExperiment/doAnalysis pipeline that fits the calibration
    regression and computes sample concentrations
  repo: https://github.com/hankemeierlab/mzQuality
- name: R
  role: Runtime environment for running mzQuality (R package, version 4.0 or later
    required)
- name: SCIEX OS 2.1.6.59781
  role: Upstream peak integration of dMRM data that supplies the area values fed into
    mzQuality
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:dff78db8c0ba73c68b5247e257de8544e5736f9af3708d369744d2ddc87ff2b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - SCIEX OS 2.1.6.59781
  - mzQuality
  edam_operation: http://edamontology.org/operation_3215
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# metabolite-concentration-quantification

## Summary

Computes absolute metabolite concentrations from targeted MS peak areas by fitting a (weighted) linear regression of compound/internal-standard ratio versus known concentration on calibration-line samples, then applying the fitted model to all other samples via mzQuality's doAnalysis.

## When to use

Use when targeted LC-MS/MS data include calibration-line samples with known spike concentrations in a 'concentration' column and you need absolute concentrations (not just ratios) for study samples.

## When NOT to use

- The dataset has no calibration-line samples with known concentrations — mzQuality then only computes ratios, batch correction and QC metrics, not absolute concentrations.
- More than one calibration sample type is present — the linear fit is restricted to a single calibration sample type, so multi-type calibration is out of scope.
- Peak integration has not been performed upstream (mzQuality processes peak-integrated data, not raw vendor files).

## Inputs

- Tab-delimited peak-integrated table with mandatory columns: aliquot, compound, area, type, injection_time, batch (and compound_is/area_is for internal standards)
- A 'concentration' column holding known spike concentrations for calibration-line samples (NA for study samples)
- A single calibration sample type (e.g. ACAL or CAL) among the sample types

## Outputs

- Calculated concentrations of compounds in all non-calibration samples
- Corresponding R2 value for the calibration regression
- Exported concentration table (e.g. concentrations.csv) via createReports

## How to apply

Load the tab-delimited data (with columns aliquot, compound, area, type, injection_time, batch, plus a concentration column) with mzQuality's readData, confirming calibration samples carry known spike concentrations while study samples hold NA, and that only a single calibration sample type (e.g. ACAL or CAL) is present. Build the SummarizedExperiment with buildExperiment, which pairs each compound with its stable isotope labeled internal standard via compound_is/area_is columns. Call doAnalysis on the experiment: it fits the (weighted) linear regression of compound/IS ratio versus known concentration on the calibration samples — restricted to one calibration sample type — and simultaneously computes the corresponding R2 value. The fitted regression is then used to calculate the concentration of the compounds in all other samples, with results added to the assay/rowData/colData slots (re-running doAnalysis overwrites prior values). Export the computed concentration table (e.g. concentrations.csv) via createReports.

## Related tools

- **mzQuality** (Provides readData/buildExperiment/doAnalysis pipeline that fits the calibration regression and computes sample concentrations) — https://github.com/hankemeierlab/mzQuality
- **R** (Runtime environment for running mzQuality (R package, version 4.0 or later required))
- **SCIEX OS 2.1.6.59781** (Upstream peak integration of dMRM data that supplies the area values fed into mzQuality)

## Examples

```
path <- system.file("extdata", "example.tsv", package = "mzQuality"); exp <- doAnalysis(buildExperiment(readData(path)))
```

## Evaluation signals

- The regression is restricted to exactly one calibration sample type; if multiple types are present the fit must be re-checked.
- Calibration samples must carry non-NA concentrations while study samples hold NA in the concentration column.
- The corresponding R2 value is reported alongside the concentrations — inspect it to judge calibration-line linearity.
- Calling doAnalysis again overwrites previous concentration values in assay/rowData/colData; verify the latest call was intended.
- Exported concentrations table should match the non-calibration sample set after the exp[rowData(exp)$use, exp$use] subsetting filter.

## Limitations

- Concentration calculation is automatic only when calibration-line samples and known concentrations are supplied; without them mzQuality does not compute absolute concentrations.
- Any NA value is treated as an absent data point and will not be used for the calculations.
- Only a single calibration sample type is supported for the fit.
- Compounds not detected in LQC or SQC samples are assumed not present and will not be reported.

## Evidence

- [methods] When calibration line samples and known concentrations for spiked compounds are supplied, mzQuality automatically calculates the concentration in all other samples during doAnalysis, based on a linear model of the ratio and known concentration.: [evidence span withheld — anchor status: not_found]
- [methods] the readData function, which verifies the required columns (aliquot, compound, area, type, injection_time, batch) plus the concentration column: [evidence span withheld — anchor status: not_found]
- [readme] If known concentrations for calibration lines have been supplied, the `doAnalysis` function will also calculate the concentrations and the corresponding R2 value: [evidence span withheld — anchor status: external_doc]
- [abstract] It processes peak-integrated data independently of vendor software and provides essential quality control features, including batch correction, outlier detection: "It processes peak-integrated data independently of vendor software and provides essential quality control features, including batch correction, outlier detection"
- [results] Any NA value is treated as an absent data point and will not be used for the calculations: "Any NA value is treated as an absent data point and will not be used for the calculations"
