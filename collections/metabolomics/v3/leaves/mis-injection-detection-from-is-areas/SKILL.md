---
name: mis-injection-detection-from-is-areas
description: Use when reviewing targeted LC-MS metabolomics data from multiple batches
  and study samples must be screened for mis-injections or low/high total IS-area
  outliers before compound-level reporting and batch correction, with flagged samples
  excluded from further calculations.
when_to_use_negative:
- Screening pooled SQC samples for outliers — those are assumed identical and should
  instead be flagged via Compound/IS ratio deviations (Rosner's test), not IS areas.
- The input is not peak-integrated targeted metabolomics data with the mandatory columns
  (aliquot, compound, area, type, injection_time, batch); mzQuality will not accept
  it.
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: mzQuality
  role: R package that loads the peak table, computes compound/IS ratios, flags study-sample
    outliers from IS areas via the Rosner test, and excludes flagged samples from
    further calculations.
  repo: https://github.com/hankemeierlab/mzQuality
- name: mzQualityDashboard
  role: Interactive Shiny dashboard recommended for inspecting mzQuality's sample
    outlier flags and IS-area plots.
  repo: https://github.com/hankemeierlab/mzQualityDashboard
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:5da7e742ba1ab9c503fd4466000278d9ae95544083ba247ec46848e6a7fb1f87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - mzQuality
  - mzQualityDashboard
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Mis-injection detection from IS areas

## Summary

Flag study-sample outliers such as probable mis-injections by inspecting Internal Standard (IS) areas across batches, following mzQuality's approach that distinguishes study samples from pooled QC samples. Because pooled SQC samples are assumed identical, QC outliers are instead detected from Compound/IS ratio deviations (via Rosner's test), whereas study samples cannot be compared against a known composition and rely on IS-area aberrations.

## When to use

Use when reviewing targeted LC-MS metabolomics data from multiple batches and study samples must be screened for mis-injections or low/high total IS-area outliers before compound-level reporting and batch correction, with flagged samples excluded from further calculations.

## When NOT to use

- Screening pooled SQC samples for outliers — those are assumed identical and should instead be flagged via Compound/IS ratio deviations (Rosner's test), not IS areas.
- The input is not peak-integrated targeted metabolomics data with the mandatory columns (aliquot, compound, area, type, injection_time, batch); mzQuality will not accept it.

## Inputs

- Tab-delimited peak-integrated LC-MS table with mandatory columns aliquot, compound, area, type, injection_time, and batch (readable via mzQuality readData)
- SummarizedExperiment object constructed by buildExperiment

## Outputs

- Sample outlier flags (colData 'use' TRUE/FALSE per sample) marking probable mis-injections or IS-area aberrations
- Flagged samples automatically excluded from further mzQuality calculations
- Violin/compound plots and reports (via createReports) for inspecting IS-area distributions across samples and batches

## How to apply

Load the tab-delimited peak table with mzQuality's readData function, which verifies the mandatory columns aliquot, compound, area, type, injection_time, and batch, then build the SummarizedExperiment object with buildExperiment and run doAnalysis to compute compound/IS ratios for every sample across the batches. For pooled QC samples, outliers are identified from the median Compound/IS ratios (since SQC samples are assumed identical) using Rosner's test; for study samples, only the Internal Standard areas are informative, so flags are triggered by abnormally low or high total IS areas, as exemplified by sample 209 with a notably lower median area than other samples. mzQuality flags potential outliers and automatically excludes them from further calculations; the analyst must then judge whether flagged study-sample outliers arise from a technical or biological source. Sample-level reliability is recorded in the colData 'use' column of the SummarizedExperiment, where FALSE indicates the sample is not reliable for reporting, and these values can be overridden manually.

## Related tools

- **mzQuality** (R package that loads the peak table, computes compound/IS ratios, flags study-sample outliers from IS areas via the Rosner test, and excludes flagged samples from further calculations.) — https://github.com/hankemeierlab/mzQuality
- **mzQualityDashboard** (Interactive Shiny dashboard recommended for inspecting mzQuality's sample outlier flags and IS-area plots.) — https://github.com/hankemeierlab/mzQualityDashboard

## Examples

```
exp <- buildExperiment(readData(path)); exp <- doAnalysis(exp = exp); exp$use
```

## Evaluation signals

- Verify that readData accepted the file — it will error if any mandatory column (aliquot, compound, area, type, injection_time, batch) is missing or incorrectly formatted.
- Confirm the SummarizedExperiment colData contains a 'use' column with TRUE/FALSE per sample reflecting outlier status.
- Check that flagged study samples (e.g. abnormally low median IS area, as in sample 209) are automatically excluded from downstream batch-corrected ratio calculations.
- Ensure the flag applies only to study samples — pooled QC outlier detection should have been performed on median Compound/IS ratios instead.

## Limitations

- Flagged study-sample outliers must be manually attributed by the user as technical or biological in origin; mzQuality does not make this distinction automatically.
- Any NA value is treated as an absent data point and will not be used for the calculations.
- Outlier decisions depend on user-set thresholds in doAnalysis; samples can also be manually overruled via the 'use' column.
- If a compound is not detected in LQC or SQC samples, it is assumed to not be present and will not be reported by mzQuality.

## Evidence

- [results] study sample mis-injections from Internal Standard areas only: [evidence span withheld — anchor status: not_found]
- [results] flagged outliers excluded from downstream calculations: "Potential outliers are flagged by mzQuality and are automatically excluded from further calculations"
- [results] sample 209 flagged for lower median area: "sample 209 in yellow, showing a notably lower area compared with the other samples in panel A"
- [methods] mandatory columns verified by readData: "you can use the `readData` function. This function will read in your data and check for any missing or incorrect columns."
- [readme] 'use' column in colData/rowData: [evidence span withheld — anchor status: external_doc]
- [readme] Rosner test for statistical outliers in QC samples: [evidence span withheld — anchor status: external_doc]
