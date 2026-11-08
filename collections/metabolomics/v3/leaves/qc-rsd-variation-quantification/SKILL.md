---
name: qc-rsd-variation-quantification
description: Use when targeted LC-MS peak-area tables include pooled QC samples (SQC/LQC)
  and you need per-compound variation metrics to flag unreliable compounds or verify
  batch-correction quality.
when_to_use_negative:
- Data contains no pooled quality control samples (SQC/LQC) — RSDqc cannot be computed
- The goal is raw peak integration rather than post-integration quality assessment;
  mzQuality processes peak-integrated data independently of vendor software
- You need de novo peak detection or feature alignment (e.g., with OpenMS or MZmine)
  rather than variation-based compound filtering
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzQuality
  role: Computes RSDqc and assigns compound quality tiers; distributed as an R package
    requiring R ≥ 4.0
  repo: https://github.com/hankemeierlab/mzQuality
- name: R
  role: Runtime environment for mzQuality (R version 4.0 or later) and downstream
    R-compatible statistical analysis of RSD outputs
- name: R-Shiny
  role: Interactive interface (mzQualityDashboard) recommended for new users to run
    RSD-based quality assessment without programming
  repo: https://github.com/hankemeierlab/mzQualityDashboard
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:cba82d6d7bbc400e5d4d091c0a928ec3f21841c71f0333bd5316452292a89e12
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - R-Shiny
  - mzQuality
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# QC RSD variation quantification

## Summary

Quantify technical variation of compounds in targeted MS metabolomics data by computing the relative standard deviation (RSD) of quality control samples (RSDqc), then use those values to flag compounds exceeding the 30% RSD threshold used by mzQuality.

## When to use

Use when targeted LC-MS peak-area tables include pooled QC samples (SQC/LQC) and you need per-compound variation metrics to flag unreliable compounds or verify batch-correction quality.

## When NOT to use

- Data contains no pooled quality control samples (SQC/LQC) — RSDqc cannot be computed
- The goal is raw peak integration rather than post-integration quality assessment; mzQuality processes peak-integrated data independently of vendor software
- You need de novo peak detection or feature alignment (e.g., with OpenMS or MZmine) rather than variation-based compound filtering

## Inputs

- Tab-delimited peak-area table (e.g., example.tsv with 105 compounds and 584 samples) with mandatory columns read by readData
- SummarizedExperiment object built with buildExperiment containing peak areas, internal standard assignments, and sample-type annotations (SQC/LQC, blanks, study samples)

## Outputs

- Per-compound RSDqc values before and after batch correction
- Compound quality tier assignments (High Confidence / With Caution / Low Signal to Noise) based on RSDqc and background-signal thresholds
- A use column in rowData (TRUE/FALSE) indicating whether each compound is reliable for reporting

## How to apply

Compute the relative standard deviation of each compound across quality control samples (RSDqc) from the peak-area or ratio values in the SummarizedExperiment built with mzQuality's buildExperiment. Run the mzQuality analysis pipeline via doAnalysis, which calculates the QC ratio and organizes compounds into quality tiers using the default thresholds: RSDqc < 15% = High Confidence, 15–30% = With Caution, and RSDqc > 30% = Low Signal to Noise, following the metabolomics QA/QC consortium guideline. Compare RSDqc before and after batch correction — a successful between-batch correction (applied on pooled study QC samples via SQC ratio deviations) should reduce QC RSD values, and validation succeeds when the post-correction PCA shows SQC samples aligned with overlapping confidence ellipses. Note that flagged potential outliers are automatically excluded from the calculations, and any NA value is treated as an absent data point and is not used.

## Related tools

- **mzQuality** (Computes RSDqc and assigns compound quality tiers; distributed as an R package requiring R ≥ 4.0) — https://github.com/hankemeierlab/mzQuality
- **R** (Runtime environment for mzQuality (R version 4.0 or later) and downstream R-compatible statistical analysis of RSD outputs)
- **R-Shiny** (Interactive interface (mzQualityDashboard) recommended for new users to run RSD-based quality assessment without programming) — https://github.com/hankemeierlab/mzQualityDashboard

## Examples

```
path <- system.file("extdata", "example.tsv", package = "mzQuality"); exp <- doAnalysis(buildExperiment(readData(path)))
```

## Evaluation signals

- Post-batch-correction RSDqc values are reduced relative to pre-correction values, with compounds meeting the RSDqc ≤ 30% quality threshold
- Compounds are partitioned into the three mzQuality tabs matching the 15% and 30% RSD cutoffs (High Confidence <15%, With Caution 15–30%, Low SNR >30%)
- The use column in rowData matches the automatic threshold decision for every compound (TRUE only where RSDqc and background-signal criteria pass)
- Potential outliers flagged by the Rosner Test on QC samples are excluded from RSD calculations
- QC violin plots and PCA plots after correction show SQC samples aligned and mostly centered within study samples with overlapping CI ellipses

## Limitations

- Compounds not detected in LQC or SQC samples are assumed absent and not reported by mzQuality
- Any NA value is treated as an absent data point and is excluded from calculations
- The 30% RSDqc and 40% background-signal thresholds are recommendations from the metabolomics QA/QC consortium and may need adjustment for other data types
- Potential outliers are automatically excluded from calculations, which can change RSDqc values depending on the outlier set

## Evidence

- [abstract] Compound quality is assessed via RSD of QC samples and compound confidence tiers: "Compound quality is further assessed by evaluating the relative standard deviation of quality control samples and the background signal from blank samples. Based on these quality metrics, compounds"
- [methods] mzQuality is an R package requiring R >= 4.0: "mzQuality is distributed as an R package for which version 4.0 or later is required."
- [intro] RSDqc threshold set at 30% per metabolomics QA/QC consortium guidelines: "We defined the threshold at a relative standard deviation of QC > 30%, which follows the guidelines of the metabolomics quality assurance"
- [results] Compounds organized into three quality tabs: High confidence, With Caution, Low Signal to Noise: "Compounds are organized into three tabs based on quality and user-defined or default settings"
- [results] NA values excluded from calculations: "Any NA value is treated as an absent data point and will not be used for the calculations"
- [readme] README: quality control features include outlier detection, batch-correction using SQC, and compound filters: "It features outlier detection, batch-correction using pooled study quality control samples (SQC), filters for removing unreliable compounds, various plots for inspecting"
- [results] Before/after batch correction PCA shows SQC alignment: [evidence span withheld — anchor status: not_found]
- [other] Task card: variation-metrics table comparing before/after states: [evidence span withheld — anchor status: not_found]
