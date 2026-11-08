---
name: relative-standard-deviation-quality-assessment
description: Use when you have raw or signal-drift-corrected LC/GC-MS metabolomics
  or proteomics feature tables with repeated pooled QC injections and need a quantitative
  precision metric (e.g. % peaks within 15% or 30% RSD) to judge correction performance.
when_to_use_negative:
- Datasets without repeated pooled QC injections throughout the batch — RSD across
  QCs is undefined and other QC-free assessment (e.g. statTarget shiftCor_dQC with
  Combat) is required instead.
- When the goal is differential-abundance or biological interpretation rather than
  quantifying technical precision — RSD on pooled QCs measures drift correction quality,
  not biological effect.
- Single-batch data where drift is negligible; the metric adds little value when raw
  QC RSD is already low.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: statTarget
  role: R package providing shiftCor (QC-RFSC, QC-RLSC, QC-SVR, Combat) and statAnalysis;
    the corrected QC table (shift_QC_cor.csv) it emits is the direct input for the
    RSD-based precision assessment
  repo: https://github.com/13479776/statTarget
- name: XCMS
  role: Used upstream to extract peak abundances from LC-MS mzXML data, producing
    the feature table on which QC RSDs are computed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.aca.2018.08.002
    title: 'statTarget: A streamlined tool for signal drift correction and interpretations
      of quantitative mass spectrometry-based omics data'
  source_content_hash: sha256:7b8ab72c489278a606ad7bc9789586eba63211ddc0f18ec7153ded457ea847b4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/13479776/statTarget
  tools:
  - XCMS
  - statTarget
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# relative-standard-deviation-quality-assessment

## Summary

Computes relative standard deviation (RSD) distributions of pooled-QC peak intensities across a batch to quantify drift-correction quality in LC/GC-MS metabolomics and proteomics feature tables.

## When to use

Use when you have raw or signal-drift-corrected LC/GC-MS metabolomics or proteomics feature tables with repeated pooled QC injections and need a quantitative precision metric (e.g. % peaks within 15% or 30% RSD) to judge correction performance.

## When NOT to use

- Datasets without repeated pooled QC injections throughout the batch — RSD across QCs is undefined and other QC-free assessment (e.g. statTarget shiftCor_dQC with Combat) is required instead.
- When the goal is differential-abundance or biological interpretation rather than quantifying technical precision — RSD on pooled QCs measures drift correction quality, not biological effect.
- Single-batch data where drift is negligible; the metric adds little value when raw QC RSD is already low.

## Inputs

- Feature-intensity table (metabolomics or proteomics) from LC/GC-MS processing (e.g. XCMS or Skyline extraction), with pooled QC samples injected repeatedly throughout the batch
- Sample annotation/metadata mapping sample IDs to QC vs biological samples and batch labels
- Corrected feature table from a signal-drift correction method (e.g. statTarget shiftCor QC-RFSC output) for post-correction evaluation

## Outputs

- Per-feature RSD values across pooled QC samples
- Cumulative RSD distribution (proportion of peaks within 15% and 30% RSD windows)
- Before/after correction comparison (e.g. fold-change in % peaks under 15% RSD, raw vs corrected)

## How to apply

For each feature, compute the relative standard deviation of its intensity across the repeated pooled QC samples in the batch (raw or corrected values). Build the cumulative RSD distribution over all features and report the proportion of peaks falling within 15% and 30% RSD cutoffs. Compare these proportions before and after a drift-correction method (e.g. QC-RFSC vs QC-RLSC vs QC-SVR vs raw) — in the article the proportion of peaks with RSD < 30% rose from 43.9% to 86.2% after QC-RFSC on dataset 1, and from 52.1% to 90.9% on dataset 2, outperforming QC-RLSC (65.5%/80.1%) and QC-SVR (61.9%/72.3%). Acceptance criterion: the corrected data should show a substantial and preferably larger increase in the fraction of peaks within the 30% RSD window than alternative correction algorithms, and a large rise in the 15% RSD window (e.g. a 12.7-fold increase to 25.1% on dataset 1). Use statTarget's shiftCor/statAnalysis output tables as the source of corrected intensities when applying this metric within the published pipeline.

## Related tools

- **statTarget** (R package providing shiftCor (QC-RFSC, QC-RLSC, QC-SVR, Combat) and statAnalysis; the corrected QC table (shift_QC_cor.csv) it emits is the direct input for the RSD-based precision assessment) — https://github.com/13479776/statTarget
- **XCMS** (Used upstream to extract peak abundances from LC-MS mzXML data, producing the feature table on which QC RSDs are computed)

## Examples

```
R: shiftCor(samPeno, samFile, Frule = 0.8, MLmethod = "QCRFSC", QCspan = 0, imputeM = "KNN") then compute RSD across shift_QC_cor.csv columns; statAnalysis(file, Frule = 0.8, normM = "NONE", imputeM = "KNN", glog = TRUE, scaling = "Pareto")
```

## Evaluation signals

- Proportion of peaks with RSD below 30% should increase substantially after correction (article: 43.9% → 86.2% on dataset 1; 52.1% → 90.9% on dataset 2).
- Proportion of peaks within 15% RSD should rise sharply (article: 1.97% raw → 25.1% after QC-RFSC, a 12.7-fold increase).
- Comparison against competing algorithms (QC-RLSC, QC-SVR) should show QC-RFSC achieving a higher fraction of peaks within the 30% RSD window.
- RSD is computed only on pooled QC sample intensities, not on biological samples — verify the input table separates QC from sample rows.
- Median and first-quartile R-squared / predictive accuracy of the correction model should be reported alongside RSD to confirm correction stability.

## Limitations

- RSD-based precision assessment depends entirely on having pooled QC samples injected at regular intervals (every 5, 6, or 8 real samples in the article's datasets 1–3); it cannot be applied to QC-free datasets.
- The metric evaluates correction quality only within the observed QC RSD distribution; it does not directly measure biological group separation or cross-batch integration accuracy.
- Reported proportions depend on the number of QC injections and batch design, so absolute values are not directly comparable across studies with different QC frequencies.

## Evidence

- [results] 30% RSD acceptance criterion for correction quality: [evidence span withheld — anchor status: not_found]
