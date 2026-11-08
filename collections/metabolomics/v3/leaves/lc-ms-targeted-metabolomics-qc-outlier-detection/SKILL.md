---
name: lc-ms-targeted-metabolomics-qc-outlier-detection
description: Use when evaluating targeted LC-MS metabolomics peak tables (tab-delimited
  with aliquot, compound, area, type, injection_time, batch columns) to identify mis-injections
  and pooled-QC outliers before batch correction or statistical analysis.
when_to_use_negative:
- Input is raw mass-spectrometry signal (mzML or vendor raw files) rather than peak-integrated
  area data — mzQuality processes peak tables and is not a peak-detection tool like
  OpenMS or MZmine.
- The goal is batch correction or compound quality assessment rather than sample outlier
  flagging — use mzQuality's doAnalysis batch-correction and RSDqc/background-signal
  filters instead.
- Compounds lack stable-isotope-labeled internal standards; study-sample outliers
  would then rely on IS-area logic that cannot be computed.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzQuality
  role: Implements readData, buildExperiment, doAnalysis and the Rosner-test-based
    outlier flagging used by this skill; flags QC and study-sample outliers and excludes
    them from further calculations.
  repo: https://github.com/hankemeierlab/mzQuality
- name: R
  role: Runtime environment; mzQuality is distributed as an R package requiring R
    4.0 or later and exposes the outlier workflow via R function calls.
  repo: https://github.com/hankemeierlab/mzQuality
- name: mzQualityDashboard
  role: Interactive Shiny front-end recommended for users who prefer a GUI over the
    package-level functions for outlier inspection and flagging.
  repo: https://github.com/hankemeierlab/mzQualityDashboard
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:777599d04c00a8f531164f64745f439ea0c6fe38fb4785ab9b9bb39bbc95a7bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - mzQuality
  - mzQualityDashboard
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# lc-ms-targeted-metabolomics-qc-outlier-detection

## Summary

Detects statistical sample outliers in targeted LC-MS metabolomics data using mzQuality's Rosner test, distinguishing pooled QC outliers (deviating compound/IS ratios) from study-sample outliers (abnormal internal standard areas), and automatically excludes flagged samples from downstream calculations.

## When to use

Use when evaluating targeted LC-MS metabolomics peak tables (tab-delimited with aliquot, compound, area, type, injection_time, batch columns) to identify mis-injections and pooled-QC outliers before batch correction or statistical analysis.

## When NOT to use

- Input is raw mass-spectrometry signal (mzML or vendor raw files) rather than peak-integrated area data — mzQuality processes peak tables and is not a peak-detection tool like OpenMS or MZmine.
- The goal is batch correction or compound quality assessment rather than sample outlier flagging — use mzQuality's doAnalysis batch-correction and RSDqc/background-signal filters instead.
- Compounds lack stable-isotope-labeled internal standards; study-sample outliers would then rely on IS-area logic that cannot be computed.

## Inputs

- Tab-delimited peak-integrated area table with mandatory columns: aliquot, compound, area, type, injection_time, batch (e.g. SCIEX OS-integrated dMRM metabolite and stable-isotope-labeled IS areas)
- SummarizedExperiment object (optionally pre-built) for the mzQuality experiment

## Outputs

- Set of flagged outlier samples (QC ratio outliers and study-sample IS-area outliers) recorded in the colData of the SummarizedExperiment
- A `use` TRUE/FALSE column in colData marking which samples are excluded from further calculations

## How to apply

Load the tab-delimited peak table with mzQuality's readData, which verifies mandatory columns (aliquot, compound, area, type, injection_time, batch), then build a SummarizedExperiment with buildExperiment and run doAnalysis, which computes compound-to-IS ratios for every sample. mzQuality flags pooled SQC outliers from deviations in the median Compound/IS ratio (QC samples are assumed identical) and flags study-sample outliers using the Internal Standard area alone, applying Rosner's test for statistical outliers; e.g. sample 209 was flagged for a notably lower median IS area and was judged a technical rather than biological variation because its deviation could be corrected by the IS ratio. Flagged outliers are automatically excluded from further calculations via a `use` column (TRUE/FALSE) in rowData/colData; users decide whether flagged study-sample outliers are technical or biological before removing them.

## Related tools

- **mzQuality** (Implements readData, buildExperiment, doAnalysis and the Rosner-test-based outlier flagging used by this skill; flags QC and study-sample outliers and excludes them from further calculations.) — https://github.com/hankemeierlab/mzQuality
- **R** (Runtime environment; mzQuality is distributed as an R package requiring R 4.0 or later and exposes the outlier workflow via R function calls.) — https://github.com/hankemeierlab/mzQuality
- **mzQualityDashboard** (Interactive Shiny front-end recommended for users who prefer a GUI over the package-level functions for outlier inspection and flagging.) — https://github.com/hankemeierlab/mzQualityDashboard

## Examples

```
path <- system.file("extdata", "example.tsv", package = "mzQuality"); exp <- doAnalysis(buildExperiment(readData(path)))
```

## Evaluation signals

- flagged_samples list exists and its entries correspond to samples whose `use` column in colData is FALSE after doAnalysis.
- Pooled SQC outliers show a compound/IS ratio deviating from the batch median, while flagged study samples (e.g. sample 209) show a notably lower median IS area relative to other samples in the batch.
- Downstream mzQuality calculations (batch-corrected ratios, RSDqc, e-score) exclude flagged samples; rerunning doAnalysis after removing flags does not change the remaining metrics.
- readData reported no missing or incorrect mandatory columns (aliquot, compound, area, type, injection_time, batch) before buildExperiment succeeded.

## Limitations

- Study-sample outlier detection uses only the Internal Standard area, so it cannot by itself distinguish a technical mis-injection from a genuine biological difference — users must make that call (as for sample 209).
- Flagged outliers are automatically excluded from further calculations, so a wrongly flagged sample must be manually overridden by setting `use` to TRUE.
- Samples with missing values (NA) will not have a data point shown and NA values are treated as absent data points, not used for calculations, which can mask outlier signals.
- Any NA value is treated as an absent data point and will not be used for the calculations, so intermittent missingness may bias outlier statistics.

## Evidence

- [abstract] outlier detection via Rosner test on QC samples and study-sample IS areas: "provides essential quality control features, including batch correction, outlier detection, and background signal assessment"
- [readme] readData validates mandatory columns before building the experiment: [evidence span withheld — anchor status: not_found]
- [readme] doAnalysis computes compound/IS ratios and related metrics on the experiment object: [evidence span withheld — anchor status: not_found]
- [readme] sample outlier selection is based on the Rosner Test over QC samples: [evidence span withheld — anchor status: external_doc]
- [results] sample 209's variation is attributable to a technical source via IS correction: "209 can be corrected by using IS, making the variation most likely from a technical source"
- [results] flagged outliers are automatically excluded from downstream calculations: "Potential outliers are flagged by mzQuality and are automatically excluded from further calculations"
- [readme] use column marks samples/compounds as reliable per thresholds: [evidence span withheld — anchor status: external_doc]
