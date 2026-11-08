---
name: batch-correction-peak-area-ratio-normalization
description: Use when targeted LC-MS peak-integrated data (e.g. SCIEX OS dMRM exports)
  show between-batch drift in median ratios across batches and need correction via
  stable-isotope internal standards before downstream analysis.
when_to_use_negative:
- Data are already batch-corrected or the goal is peak integration rather than post-integration
  quality correction — mzQuality processes peak-integrated data independently of vendor
  software and does not re-process raw peaks.
- Only one batch is present and no pooled SQC samples were measured, so between-batch
  adjustment is undefined.
- The aim is spectral processing (baseline noise filtering, retention time alignment)
  — tools like OpenMS or MZmine are the appropriate choice, not mzQuality.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0154
tools:
- name: mzQuality
  role: R package providing readData, buildExperiment, doAnalysis (which computes
    compound/IS ratios and performs SQC-based batch correction), and pcaPlot for validating
    the correction.
  repo: https://github.com/hankemeierlab/mzQuality
- name: mzQualityDashboard
  role: Shiny dashboard front-end recommended for interactive use of the same batch-correction
    workflow.
  repo: https://github.com/hankemeierlab/mzQualityDashboard
- name: SCIEX OS
  role: Vendor software used upstream to integrate peak areas that mzQuality then
    batch-corrects; mzQuality does not itself re-process peaks.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:543579ebbae1ea15bbd4d5f134c210c21e3e4006b1ebdb79621de587e22d5767
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - SCIEX OS
  - mzQuality
  - mzQualityDashboard
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# batch-correction-peak-area-ratio-normalization

## Summary

Batch-corrects targeted LC-MS peak areas by computing compound/internal-standard ratios and adjusting between batches using pooled study quality control (SQC) samples, as implemented in the mzQuality R package.

## When to use

Use when targeted LC-MS peak-integrated data (e.g. SCIEX OS dMRM exports) show between-batch drift in median ratios across batches and need correction via stable-isotope internal standards before downstream analysis.

## When NOT to use

- Data are already batch-corrected or the goal is peak integration rather than post-integration quality correction — mzQuality processes peak-integrated data independently of vendor software and does not re-process raw peaks.
- Only one batch is present and no pooled SQC samples were measured, so between-batch adjustment is undefined.
- The aim is spectral processing (baseline noise filtering, retention time alignment) — tools like OpenMS or MZmine are the appropriate choice, not mzQuality.

## Inputs

- tab-delimited text file with mandatory columns aliquot, compound, area, type, injection_time, batch (or an existing SummarizedExperiment)
- example dataset shipped with mzQuality (inst/extdata/example.tsv; 105 compounds, 584 samples over six batches)
- pooled study quality control (SQC) samples measured across batches

## Outputs

- batch-corrected compound/IS ratios stored in the 'ratio_corrected' assay of the SummarizedExperiment
- updated rowData and colData slots with calculated metrics
- PCA plot of batch-corrected ratios per aliquot type and batch (via pcaPlot)
- optional report folder from createReports containing HTML plots, tab-delimited assays, and an Excel summary

## How to apply

Read the tab-delimited data with mzQuality's readData (which checks mandatory columns: aliquot, compound, area, type, injection_time, batch) and build the SummarizedExperiment object with buildExperiment. Compute the ratio between each compound and its assigned internal standard, then perform batch correction using pooled study quality control (SQC) samples — mzQuality applies a between-batch adjustment of these ratios so that SQC samples from different batches align. Potential outliers flagged by mzQuality are automatically excluded from further calculations, and any NA value is treated as an absent data point and will not be used for the calculations. Validate the correction by inspecting a PCA plot of batch-corrected ratios (pcaPlot with assay='ratio_corrected') to confirm SQC samples from different batches are aligned and mostly centered with overlapping confidence-interval ellipses. Batch correction is one step of the doAnalysis wrapper, which also computes background signal, matrix effect, QC ratio, compound presence, median area, and internal standard suggestions.

## Related tools

- **mzQuality** (R package providing readData, buildExperiment, doAnalysis (which computes compound/IS ratios and performs SQC-based batch correction), and pcaPlot for validating the correction.) — https://github.com/hankemeierlab/mzQuality
- **mzQualityDashboard** (Shiny dashboard front-end recommended for interactive use of the same batch-correction workflow.) — https://github.com/hankemeierlab/mzQualityDashboard
- **SCIEX OS** (Vendor software used upstream to integrate peak areas that mzQuality then batch-corrects; mzQuality does not itself re-process peaks.)

## Examples

```
path <- system.file("extdata", "example.tsv", package = "mzQuality"); exp <- doAnalysis(buildExperiment(readData(path))); pcaPlot(exp, assay = "ratio_corrected")
```

## Evaluation signals

- PCA plot of the 'ratio_corrected' assay shows SQC samples from different batches aligned and mostly centered with overlapping CI ellipses, whereas pre-correction they were not aligned.
- Deviations in the median ratio at batch boundaries (e.g. last two samples of batch 4 and first three of batch 5) are removed after the between-batch adjustment.
- The 'ratio_corrected' assay is populated in the SummarizedExperiment after doAnalysis; calling doAnalysis again overwrites previous values, so re-running yields consistent results.
- Outliers flagged by mzQuality and NA values are excluded from the correction — no flagged outlier should contribute to the corrected ratios.
- Corrected ratios for a sample with a technically low median area (e.g. sample 209) track the rest of the batch after correction via IS, confirming technical (not biological) variation was removed.

## Limitations

- Compounds not detected in LQC or SQC samples are assumed absent and not reported by mzQuality, so they cannot be batch-corrected.
- Any NA value is treated as an absent data point and is not used in the calculations, leaving gaps in coverage.
- Batch correction relies on pooled SQC samples; without them the between-batch adjustment cannot be performed.
- The doAnalysis wrapper recomputes and overwrites all assays, so prior manual adjustments to assay slots are lost on re-run.

## Evidence

- [abstract] mzQuality processes peak-integrated data independently of vendor software and provides batch correction, outlier detection, background signal assessment, and trend visualization.: "It processes peak-integrated data independently of vendor software and provides essential quality control features, including batch correction, outlier detection, and background signal assessment"
- [readme] doAnalysis wrapper steps include computing compound/IS ratios and performing SQC-based batch correction.: "1.  Calculate the ratio between the compounds and assigned internal standards,
2.  Perform batch correction using the pooled study quality control samples (SQC),"
- [results] Batch correction aligns SQC samples across batches in PCA of corrected ratios.: [evidence span withheld — anchor status: not_found]
- [results] NA values are excluded from calculations.: "Any NA value is treated as an absent data point and will not be used for the calculations"
- [methods] mzQuality is an R package; data read via readData and converted to SummarizedExperiment by buildExperiment.: "mzQuality is distributed as an R package for which version 4.0 or later is required."
- [results] Outliers are automatically excluded from further calculations.: "Potential outliers are flagged by mzQuality and are automatically excluded from further calculations"
- [methods] readData checks mandatory columns before building the experiment.: "you can use the `readData` function. This function will read in your data and check for any missing or incorrect columns."
- [results] Missing compounds in QC samples are not reported.: "When a compound is not detected in LQC or SQC samples, the compound will be assumed to not be present and will not be reported by mzQuality"
