---
name: targeted-mrm-data-handling
description: Use when you have peak-integrated targeted MRM/dMRM LC-MS metabolomics
  data (tab-delimited or SummarizedExperiment) needing QC, batch correction, outlier
  flagging, compound filtering, or absolute concentration calculation.
when_to_use_negative:
- Input is unprocessed raw MS vendor files needing peak integration — mzQuality processes
  peak-integrated data and requires prior integration (e.g. by SCIEX OS, OpenMS, MZmine)
  rather than processing peaks itself.
- Unbiased/untargeted feature table without isotope-labeled internal standards per
  compound or pooled QC samples across batches — batch correction and ratio QC depend
  on the stable-isotope-labeled IS and SQC design.
- A compound is not detected in LQC or SQC samples — it is assumed absent and will
  not be reported, so absolute quantification on that compound is inappropriate.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzQuality
  role: R package implementing readData, buildExperiment, doAnalysis, doAnalysis calibration-line
    quantification, and createReports for targeted MRM metabolomics QC
  repo: https://github.com/hankemeierlab/mzQuality
- name: mzQualityDashboard
  role: Shiny app wrapper recommended for interactive use of mzQuality, especially
    for new R users
  repo: https://github.com/hankemeierlab/mzQualityDashboard
- name: R
  role: Runtime environment (R ≥ 4.0) for installing and running mzQuality as an R
    package
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:e4b79fe479a5a875497480202b30c4f3f639b1f18c465b98b27c028f58906c1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - mzQuality
  - mzQualityDashboard
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# targeted-mrm-data-handling

## Summary

Skill for handling targeted LC-MS/MS MRM (dynamic Multiple Reaction Monitoring) metabolomics peak-integrated data through mzQuality: reading tab-delimited peak tables, computing compound/internal-standard ratios, batch-correcting against pooled study QC samples, assessing compound quality via QC RSD and blank background signal, and optionally quantifying absolute concentrations from calibration lines.

## When to use

Use when you have peak-integrated targeted MRM/dMRM LC-MS metabolomics data (tab-delimited or SummarizedExperiment) needing QC, batch correction, outlier flagging, compound filtering, or absolute concentration calculation.

## When NOT to use

- Input is unprocessed raw MS vendor files needing peak integration — mzQuality processes peak-integrated data and requires prior integration (e.g. by SCIEX OS, OpenMS, MZmine) rather than processing peaks itself.
- Unbiased/untargeted feature table without isotope-labeled internal standards per compound or pooled QC samples across batches — batch correction and ratio QC depend on the stable-isotope-labeled IS and SQC design.
- A compound is not detected in LQC or SQC samples — it is assumed absent and will not be reported, so absolute quantification on that compound is inappropriate.

## Inputs

- Tab-delimited peak-integrated MRM table (extdata/example.tsv) with columns aliquot, compound, area, type, injection_time, batch, compound_is, area_is; concentration column for calibration lines
- SummarizedExperiment object
- Calibration-line sample rows with known spike concentrations and a single calibration sample type (e.g. ACAL or CAL)

## Outputs

- SummarizedExperiment with updated assay, rowData, and colData slots (ratio, ratio_corrected, background signal, matrix effect, median area, use flags)
- Compound quality classification: High Confidence (RSDqc <15%), With Caution (15–30%), Low Signal to Noise (>40% background or RSDqc >30%)
- Absolute concentration table (concentrations.csv) with fitted linear-regression R2 per compound
- HTML/Excel/tab-delimited report folder from createReports
- Plot objects: aliquotPlot, compoundPlot, pcaPlot (ratio_corrected), violinPlot

## How to apply

Load peak-integrated data with `readData`, which validates the mandatory columns (aliquot, compound, area, type, injection_time, batch; plus compound_is/area_is for internal standards and concentration for calibration lines), then convert to a SummarizedExperiment with `buildExperiment`. Run `doAnalysis` to compute compound/IS ratios, batch-correct the ratio using pooled study quality control (SQC) samples, calculate background signal relative to blanks, matrix effect, QC-sample ratio, compound presence, median area, and suggested internal standards. Classify compounds into High Confidence (RSDqc < 15%), With Caution (15–30%), and Low Signal to Noise (background signal e-score > 40% or RSDqc > 30%), and subset with `exp <- exp[rowData(exp)$use, exp$use]`, overriding the `use` flags manually where justified. If calibration-line samples with known spike concentrations in the `concentration` column (one calibration sample type, e.g. ACAL or CAL) are supplied, `doAnalysis` fits a (weighted) linear regression of compound/IS area ratio versus known concentration on the calibration samples and calculates absolute concentrations in all other samples. Export the report with `createReports` and downstream tables (concentrations.csv, HTML plots, Excel file) for statistical analysis.

## Related tools

- **mzQuality** (R package implementing readData, buildExperiment, doAnalysis, doAnalysis calibration-line quantification, and createReports for targeted MRM metabolomics QC) — https://github.com/hankemeierlab/mzQuality
- **mzQualityDashboard** (Shiny app wrapper recommended for interactive use of mzQuality, especially for new R users) — https://github.com/hankemeierlab/mzQualityDashboard
- **R** (Runtime environment (R ≥ 4.0) for installing and running mzQuality as an R package)

## Examples

```
exp <- buildExperiment(readData(system.file("extdata","example.tsv",package="mzQuality"))); exp <- doAnalysis(exp); exp <- exp[rowData(exp)$use, exp$use]
```

## Evaluation signals

- All mandatory columns (aliquot, compound, area, type, injection_time, batch) pass the readData check without missing or incorrect-column errors.
- After batch correction, PCA of ratio_corrected shows SQC samples aligned and centered with overlapping confidence ellipses across batches.
- Compound RSDqc threshold applied: RSDqc <15% High Confidence, 15–30% With Caution, and background signal (e-score) ≤ 40% (RSDqc >30% flags Low SNR).
- If calibration lines were supplied, fitted linear regression R2 for compound concentration is reported and concentrations.csv contains non-NA concentration values for non-calibration samples.
- Potential outliers flagged by mzQuality are automatically excluded from further calculations, and the `use` column in rowData/colData is TRUE only for retained compounds/samples.

## Limitations

- Any NA value is treated as an absent data point and will not be used for the calculations; samples with missing values will not have a data point shown in individual compound plots.
- When a compound is not detected in LQC or SQC samples, it is assumed not present and not reported by mzQuality.
- Calibration-line fitting requires only a single calibration sample type and a known spike concentration in the concentration column; concentrations and R2 are only computed when these are supplied.
- If doAnalysis is called again it will overwrite previously computed assay/rowData/colData values.
- Only one calibration sample type (e.g. ACAL or CAL) may be present per run.
- mzQuality flags potential outliers automatically and excludes them from further calculations; manual override of use flags is possible but thresholds should be justified per metabolomics QA/QC guidelines.

## Evidence

- [abstract] mzQuality processes peak-integrated data independently of vendor software and provides batch correction, outlier detection, background signal assessment, and trend visualization.: "It processes peak-integrated data independently of vendor software and provides essential quality control features, including batch correction, outlier detection, and background signal assessment"
- [readme] The `doAnalysis` function will perform the following steps: 1. Calculate the ratio between the compounds and assigned internal standards, 2. Perform batch correction using the pooled study quality control samples (SQC), 3. Calculate the percentage of background signal compared to the study samples, 4. Calculate the matrix effect,: "The `doAnalysis` function will perform the following steps: 1.  Calculate the ratio between the compounds and assigned internal standards,
2.  Perform batch correction using the pooled study quality"
- [readme] If known concentrations for calibration lines have been supplied, the `doAnalysis` function will also calculate the concentrations and the corresponding R2 value given the provided calibration lines.: "If known concentrations for calibration lines have been supplied, the 
`doAnalysis` function will also calculate the concentrations and the
corresponding R2 value given the provided calibration lines."
- [results] Compounds are organized into three tabs based on quality: High confidence (RSDqc <15%), With Caution (15–30%), Low Signal to Noise (background signal >40% or RSDqc >30%): "Compounds are organized into three tabs based on quality and user-defined or default settings"
- [readme] readData validates mandatory columns; buildExperiment creates the SummarizedExperiment used throughout the package.: "Once your files are ready, you can use the `readData` function to read in your 
data. It will check if all mandatory columns are present and if the data is in 
the correct format."
- [readme] Example dataset: 105 compounds and 584 samples shipped in extdata/example.tsv.: "an example dataset containing 
105 compounds and 584 samples has been added. These can be loaded by running 
the following code."
