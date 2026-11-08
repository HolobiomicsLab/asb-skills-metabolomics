---
name: metabolomics-missing-value-and-outlier-exclusion
description: Use when a targeted LC-MS metabolomics dataset (tab-delimited peak areas
  with aliquot, compound, area, type, injection_time, batch columns) contains NA values
  or samples flagged as statistical outliers in QC samples, and downstream calculations
  must exclude them.
when_to_use_negative:
- Input is not peak-integrated targeted metabolomics data (e.g. already a processed
  feature table from untargeted tools like OpenMS or MZmine) — mzQuality expects vendor-integrated
  peak areas from dMRM/polarity-switching LC-MS.
- A compound is entirely absent from LQC and SQC samples — mzQuality assumes it is
  not present and does not report it, so exclusion logic should not be applied to
  it.
- The goal is de novo statistical outlier detection on raw peak areas without QC sample
  context — the Rosner Test in mzQuality operates on QC samples specifically, not
  arbitrary study samples.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0219
tools:
- name: mzQuality
  role: Flags sample outliers via the Rosner Test on QC samples, excludes flagged
    outliers and NA values from calculations, and writes boolean `use` columns used
    to subset reliable samples and compounds.
  repo: https://github.com/hankemeierlab/mzQuality
- name: R
  role: Runtime environment (R >= 4.0) in which mzQuality is installed and the readData/buildExperiment/doAnalysis/plot
    pipeline is executed.
- name: mzQualityDashboard
  role: Interactive Shiny front-end over mzQuality recommended for reviewing flagged
    outliers and compounds visually before accepting the automatic use-selection.
  repo: hankemeierlab/mzQualityDashboard
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:ee68561ab6586399b848afcad48dc7539acbb8547f462ad5b2a9f97da696f62c
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
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metabolomics-missing-value-and-outlier-exclusion

## Summary

Excludes statistical outliers flagged by the Rosner test on QC samples and treats NA values as absent data points before downstream calculations in mzQuality-based targeted metabolomics QC, so that unreliable samples and missing signals do not bias metrics such as RSDQC and batch correction.

## When to use

Use when a targeted LC-MS metabolomics dataset (tab-delimited peak areas with aliquot, compound, area, type, injection_time, batch columns) contains NA values or samples flagged as statistical outliers in QC samples, and downstream calculations must exclude them.

## When NOT to use

- Input is not peak-integrated targeted metabolomics data (e.g. already a processed feature table from untargeted tools like OpenMS or MZmine) — mzQuality expects vendor-integrated peak areas from dMRM/polarity-switching LC-MS.
- A compound is entirely absent from LQC and SQC samples — mzQuality assumes it is not present and does not report it, so exclusion logic should not be applied to it.
- The goal is de novo statistical outlier detection on raw peak areas without QC sample context — the Rosner Test in mzQuality operates on QC samples specifically, not arbitrary study samples.

## Inputs

- Tab-delimited peak-integrated metabolomics data table with mandatory columns: aliquot, compound, area, type, injection_time, batch
- SummarizedExperiment object built by mzQuality's buildExperiment

## Outputs

- SummarizedExperiment with boolean `use` columns in rowData and colData indicating which compounds and samples are reliable for reporting
- Subset SummarizedExperiment excluding flagged outliers and NA-only data points from all downstream calculations and plots

## How to apply

Read the tab-delimited dataset with mzQuality's readData function, which verifies mandatory columns (aliquot, compound, area, type, injection_time, batch), then build a SummarizedExperiment with buildExperiment. Run doAnalysis to compute compound/IS ratios, SQC-based batch correction, background signal percentage, matrix effect, QC-sample ratio, compound presence, and median area. mzQuality flags potential sample outliers via the Rosner Test applied to QC samples and automatically excludes flagged outliers from further calculations; any NA value is likewise treated as an absent data point and not used for calculations. Apply the exclusions by subsetting with the boolean `use` columns in rowData and colData (e.g. exp[rowData(exp)$use, exp$use]), which can be manually overridden to TRUE/FALSE. The rationale is that outlier samples (e.g. a sample with a notably lower median area) reflect technical variation correctable by internal standards, so leaving them in would distort RSDQC, batch correction, and internal standard selection.

## Related tools

- **mzQuality** (Flags sample outliers via the Rosner Test on QC samples, excludes flagged outliers and NA values from calculations, and writes boolean `use` columns used to subset reliable samples and compounds.) — https://github.com/hankemeierlab/mzQuality
- **R** (Runtime environment (R >= 4.0) in which mzQuality is installed and the readData/buildExperiment/doAnalysis/plot pipeline is executed.)
- **mzQualityDashboard** (Interactive Shiny front-end over mzQuality recommended for reviewing flagged outliers and compounds visually before accepting the automatic use-selection.) — hankemeierlab/mzQualityDashboard

## Examples

```
path <- system.file("extdata", "example.tsv", package = "mzQuality"); exp <- doAnalysis(buildExperiment(readData(path))); exp <- exp[rowData(exp)$use, exp$use]
```

## Evaluation signals

- Verify that readData did not raise a missing-column error, confirming aliquot, compound, area, type, injection_time, and batch are present.
- Confirm the subsetted experiment dims equal the number of TRUE entries in rowData(exp)$use and exp$use respectively.
- Check that no value used in RSDQC or ratio_corrected computations originates from a Rosner-flagged outlier sample.
- Confirm samples whose compound values are NA have no data point in compoundPlot outputs.
- Cross-check that SQC samples remain aligned and centered with overlapping CI ellipses in pcaPlot on ratio_corrected after exclusion.

## Limitations

- Excluded outliers are removed from calculations rather than corrected; correction of technical variation is handled separately via internal standard ratios.
- The Rosner Test only identifies statistical outliers in QC samples, so outliers among study samples are not automatically flagged by that mechanism.
- When a compound is not detected in LQC or SQC samples, mzQuality assumes it is not present and will not report it — exclusion logic does not resurrect such compounds.
- Calling doAnalysis again overwrites previous assay, rowData, and colData values, so a re-run can discard prior manual use-overrides.

## Evidence

- [results] Potential outliers are flagged by mzQuality and are automatically excluded from further calculations: "Potential outliers are flagged by mzQuality and are automatically excluded from further calculations"
- [results] Any NA value is treated as an absent data point and will not be used for the calculations: "Any NA value is treated as an absent data point and will not be used for the calculations"
- [readme] For samples, this is based on the outcome of the Rosner Test, which tests for statistical outliers in QC samples.: [evidence span withheld — anchor status: external_doc]
- [readme] These contain either a `TRUE` or `FALSE` value, indicating if the compound or sample is reliable for reporting based on the set thresholds.: [evidence span withheld — anchor status: external_doc]
- [results] Samples with missing values, as indicated by NA, will not have a data point shown: "Samples with missing values, as indicated by NA, will not have a data point shown"
