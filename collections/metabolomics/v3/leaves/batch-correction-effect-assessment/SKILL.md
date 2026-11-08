---
name: batch-correction-effect-assessment
description: Use when targeted LC-MS peak-area data spanning multiple batches (e.g.
  dMRM MRM measurements with internal standards) has been batch-corrected and you
  must demonstrate that SQC samples align across batches and QC RSD drops before trusting
  downstream statistics.
when_to_use_negative:
- Data has not been batch-corrected yet — use mzQuality's doAnalysis first; this skill
  only assesses the effect, it does not perform correction
- Input is untargeted metabolomics peak data processed by OpenMS/MZmine rather than
  peak-integrated targeted LC-MS measurements with assigned internal standards
- Only a single batch is present — no between-batch variation exists to assess
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
tools:
- name: mzQuality
  role: Performs between-batch correction using pooled SQC samples, flags potential
    outliers via Rosner Test, and produces pcaPlot/violinPlot diagnostics on ratio
    and ratio_corrected assays for before/after comparison
  repo: https://github.com/hankemeierlab/mzQuality
- name: mzQualityDashboard
  role: Interactive Shiny interface to visualize PCA alignment and QC variation before/after
    batch correction
  repo: https://github.com/hankemeierlab/mzQualityDashboard
- name: R
  role: Runtime for mzQuality (requires R ≥ 4.0) and for custom computation of per-compound
    QC RSD before vs after correction
  repo: https://cran.r-project.org/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/jasms.5c00073
    title: 'mzQuality: An Open-Source Software Tool for Quality Monitoring and Reporting
      of Targeted Mass Spectrometry Measurements'
  source_content_hash: sha256:df0dbb121a8817f0ce6ba84a4707885f2061218154a47e6fd05ea60a7ac90f8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hankemeierlab/mzQuality
  tools:
  - R
  - R-Shiny
  - mzQuality
  - mzQualityDashboard
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# batch-correction-effect-assessment

## Summary

Assess whether mzQuality's between-batch correction, driven by pooled study quality control (SQC) samples, actually reduced technical variation in targeted LC-MS metabolomics data by comparing pre- and post-correction PCA plots, QC violin plots, and per-compound QC relative standard deviations.

## When to use

Use when targeted LC-MS peak-area data spanning multiple batches (e.g. dMRM MRM measurements with internal standards) has been batch-corrected and you must demonstrate that SQC samples align across batches and QC RSD drops before trusting downstream statistics.

## When NOT to use

- Data has not been batch-corrected yet — use mzQuality's doAnalysis first; this skill only assesses the effect, it does not perform correction
- Input is untargeted metabolomics peak data processed by OpenMS/MZmine rather than peak-integrated targeted LC-MS measurements with assigned internal standards
- Only a single batch is present — no between-batch variation exists to assess

## Inputs

- Peak-integrated targeted LC-MS peak-area table (tab-delimited, e.g. mzQuality example.tsv) with compounds and internal standards measured across multiple batches
- Pooled study quality control samples (SQC) spanning all batches
- SummarizedExperiment object built by buildExperiment

## Outputs

- PCA plot of study vs SQC samples per batch for batch-corrected ratios (pcaPlot on ratio_corrected assay)
- Violin plots of QC sample distribution before and after correction (violinPlot)
- Per-compound QC relative standard deviations before and after batch correction
- Batch-corrected ratio assay (ratio_corrected) in the SummarizedExperiment

## How to apply

Load the peak-area table (e.g. the packaged example.tsv with 105 compounds and 584 samples) with readData, build the SummarizedExperiment via buildExperiment, and generate 'before' diagnostics: a pcaPlot of study vs SQC samples per batch and violinPlot of QC distribution. Run doAnalysis, which computes compound/IS ratios and performs batch correction using the pooled SQC samples, then regenerate pcaPlot on the ratio_corrected assay to check that SQC samples from each batch align and mostly center within the study samples with overlapping confidence ellipses. Recompute the relative standard deviation of QC samples per compound before and after correction; compounds meeting the RSDqc ≤ 30% threshold (following metabolomics QA/QC consortium guidelines) should show reduced RSD after correction. Verify flagged potential outliers are excluded from calculations — mzQuality flags them automatically via the Rosner Test on QC samples, setting the use column in colData to FALSE. Accept the correction only if both the PCA alignment improves and QC RSDs are reduced; if a compound's variation is not reduced by IS-based correction, suspect a biological rather than technical source.

## Related tools

- **mzQuality** (Performs between-batch correction using pooled SQC samples, flags potential outliers via Rosner Test, and produces pcaPlot/violinPlot diagnostics on ratio and ratio_corrected assays for before/after comparison) — https://github.com/hankemeierlab/mzQuality
- **mzQualityDashboard** (Interactive Shiny interface to visualize PCA alignment and QC variation before/after batch correction) — https://github.com/hankemeierlab/mzQualityDashboard
- **R** (Runtime for mzQuality (requires R ≥ 4.0) and for custom computation of per-compound QC RSD before vs after correction) — https://cran.r-project.org/

## Examples

```
path <- system.file("extdata", "example.tsv", package = "mzQuality"); exp <- doAnalysis(buildExperiment(readData(path))); pcaPlot(exp, assay = "ratio_corrected"); violinPlot(exp)
```

## Evaluation signals

- Post-correction PCA shows SQC samples from each batch aligned and mostly centered within study samples with overlapping confidence ellipses (CI ellipses overlap across the six batches)
- Per-compound QC RSD values are reduced after correction and compounds fall at or below the RSDqc ≤ 30% threshold
- All flagged potential outliers (Rosner Test on QC samples) have use=FALSE in colData and are excluded from RSD calculations
- Compounds with background signal (e-score) ≤ 40% remain classified as High Confidence or Caution rather than Low Signal to Noise
- Median-ratio deviations at batch boundaries (e.g. last two samples of batch 4, first three of batch 5) are visibly flattened after between-batch adjustment

## Limitations

- mzQuality's correction relies on SQC samples; when a compound is not detected in LQC or SQC samples it is assumed absent and not reported
- Any NA value is treated as an absent data point and excluded from calculations, so sparse compounds may yield misleading RSD comparisons
- Batch correction corrects technical variation; if sample variation is not corrected by IS, mzQuality attributes it to a likely biological source rather than a technical one
- Requires R version 4.0 or later

## Evidence

- [results] PCA alignment after correction: [evidence span withheld — anchor status: not_found]
- [results] RSDqc threshold: "we set the threshold for the RSDqc at 30%. For the background signal, we recommend that the value of the e-score should not exceed 40%"
- [readme] doAnalysis steps: [evidence span withheld — anchor status: external_doc]
- [results] Outlier exclusion: "Potential outliers are flagged by mzQuality and are automatically excluded from further calculations"
- [readme] Example dataset: [evidence span withheld — anchor status: external_doc]
- [readme] Rosner test for samples: [evidence span withheld — anchor status: external_doc]
- [readme] pcaPlot on corrected assay: [evidence span withheld — anchor status: external_doc]
- [results] IS-based correction indicates technical source: "209 can be corrected by using IS, making the variation most likely from a technical source"
