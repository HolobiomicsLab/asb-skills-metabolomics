---
name: chromatographic-peak-quality-metric-computation
description: Use when you have an XCMS pre-processed untargeted LC–MS dataset (getEIC
  + fillPeaks output) and need numeric per-peak quality features to train or apply
  a Pass/Fail peak-quality classifier.
when_to_use_negative:
- Input is not an XCMS object — MetaClean requires an XCMS object input and is not
  amenable in its current form to uniquely formatted vendor-processed outputs such
  as MassHunter Profinder or Progenesis QI results.
- The goal is targeted-MS peak assessment with vendor-specific transition-level metrics
  rather than untargeted EIC quality metrics.
- Peaks have already been filtered (e.g. by RSD < 30% in pooled QC) and only their
  integration quality remains to be judged — the metric computation is a feature-generation
  step, not a filter.
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: XCMS
  role: Provides the pre-processed LC–MS data (peak-picking, retention time correction,
    grouping) whose getEIC/fillPeaks outputs are the input to metric computation.
- name: MetaClean
  role: Hosts the 12 peak-quality metric calculation functions (calculateApexMaxBoundaryRatio,
    calculateJaggedness, calculateSymmetry, calculateModality, calculateFWHM, calculateElutionShift,
    calculateRetentionTimeConsistency, calculateSharpness, calculateGaussianSimilarity,
    calculateTPASR, calculateZigZagIndex) and getEvalObj/getPeakQualityMetrics wrappers.
  repo: https://CRAN.R-project.org/package=MetaClean
- name: TargetedMSQC
  role: Source of the M7 metric definitions (apex-boundary ratio, jaggedness, symmetry,
    modality, FWHM2base, elution shift, retention time consistency) repurposed for
    untargeted metabolomics.
provenance:
  source_task_ids:
  - task_001
  - task_008
  - task_007
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:93c4043840406cd1e2c7049f355955a7d01943a1a88f70972a1610155d8aedbb
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IPO
  - MLmetrics
  - MetaClean
  - TargetedMSQC
  - XCMS
  - scmamp
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# chromatographic-peak-quality-metric-computation

## Summary

Computes per-peak chromatographic quality metrics (apex-boundary ratio, jaggedness, symmetry, modality, FWHM2base, elution shift, retention-time consistency, sharpness, Gaussian similarity, TPASR, zig-zag index) from an XCMS-extracted ion chromatogram (evalObj) to serve as classifier features for Pass/Fail peak-quality assessment in untargeted LC–MS metabolomics.

## When to use

Use when you have an XCMS pre-processed untargeted LC–MS dataset (getEIC + fillPeaks output) and need numeric per-peak quality features to train or apply a Pass/Fail peak-quality classifier.

## When NOT to use

- Input is not an XCMS object — MetaClean requires an XCMS object input and is not amenable in its current form to uniquely formatted vendor-processed outputs such as MassHunter Profinder or Progenesis QI results.
- The goal is targeted-MS peak assessment with vendor-specific transition-level metrics rather than untargeted EIC quality metrics.
- Peaks have already been filtered (e.g. by RSD < 30% in pooled QC) and only their integration quality remains to be judged — the metric computation is a feature-generation step, not a filter.

## Inputs

- XCMS pre-processed LC–MS dataset (xcmsEIC object from getEIC() and filled object from fillPeaks())
- evalObj object produced by getEvalObj(xs, fill) containing eicPts, eicPeakData, and eicNos
- Optional Pass/Fail labels for peaks (e.g. expert-assigned on 500 peaks of ST000726)
- flatness.factor noise threshold parameter for TargetedMSQC-derived metrics

## Outputs

- Per-peak metric matrix (500 x M; M = 4 for M4, 7 for M7, 11 for M11) for each metric set
- Metric tables in metric_tables.csv with EIC numbers and optional Pass/Fail labels as columns

## How to apply

Run getEvalObj() on an XCMS object containing the extracted ion chromatograms (from getEIC) and the filled peak data (from fillPeaks), producing eicPts, eicPeakData, and eicNos. Then apply the MetaClean metric functions to each EIC: for the M7 set (TargetedMSQC-derived) use calculateApexMaxBoundaryRatio, calculateJaggedness, calculateSymmetry, calculateModality, calculateFWHM, calculateElutionShift, and calculateRetentionTimeConsistency with the flatness.factor noise threshold; for the M4 set (Zhang et al. 2014) use calculateSharpness, calculateGaussianSimilarity, calculateTPASR, and calculateZigZagIndex. For each peak, compute every metric across all N individual samples and take the mean per sample to yield a single per-peak quality value, assembling a 500 x M matrix (M = 4, 7, 11 for M4, M7, M11). Save one metric table per set with EIC numbers (and optional Pass/Fail labels) as columns for downstream classifier training or scoring.

## Related tools

- **XCMS** (Provides the pre-processed LC–MS data (peak-picking, retention time correction, grouping) whose getEIC/fillPeaks outputs are the input to metric computation.)
- **MetaClean** (Hosts the 12 peak-quality metric calculation functions (calculateApexMaxBoundaryRatio, calculateJaggedness, calculateSymmetry, calculateModality, calculateFWHM, calculateElutionShift, calculateRetentionTimeConsistency, calculateSharpness, calculateGaussianSimilarity, calculateTPASR, calculateZigZagIndex) and getEvalObj/getPeakQualityMetrics wrappers.) — https://CRAN.R-project.org/package=MetaClean
- **TargetedMSQC** (Source of the M7 metric definitions (apex-boundary ratio, jaggedness, symmetry, modality, FWHM2base, elution shift, retention time consistency) repurposed for untargeted metabolomics.)

## Examples

```
metaclean::getPeakQualityMetrics(getEvalObj(xs, fill))  # xs: XCMS getEIC result; fill: fillPeaks result
```

## Evaluation signals

- The output metric matrix has exactly 4, 7, or 11 numeric columns (M4, M7, M11) plus EIC identifiers, and one row per peak.
- Each metric value is the mean of the per-sample metric computed across all individual samples for that peak, not a per-sample value.
- The M4 set contains exactly sharpness, Gaussian similarity, TPASR, and local zigzag index; the M7 set contains exactly apex-boundary ratio, jaggedness, symmetry, modality, FWHM2base, elution shift, and retention time consistency.
- Metric tables can be merged with Pass/Fail labels (e.g. the 500 expert-labeled peaks of ST000726) and used to train the 24 candidate classifiers (3 metric sets x 8 caret algorithms).
- Metrics computed on known low-quality peaks (Fail labels) should show degraded values (e.g. low apex-boundary ratio, high zig-zag index) relative to known high-quality (Pass) peaks, supporting separability before classifier training.

## Limitations

- MetaClean requires an XCMS object input and is not amenable to uniquely formatted vendor-processed outputs.
- Metrics depend on the characteristics of the LC–MS run — ionization mode, analytical platform and method, and biological matrix affect peak shape and size, and ultimately the quality of integrations.
- The flatness.factor noise threshold must be set for TargetedMSQC-derived metrics; it directly influences jaggedness and modality calculations on low-signal EICs.
- Mean-of-per-sample aggregation smooths intra-batch variability; peaks that are good in only a subset of samples may receive intermediate metric values and be hard to classify.

## Evidence

- [methods] M7 metric set adapted from TargetedMSQC — apex-boundary ratio, jaggedness, symmetry, modality, FWHM2base, elution shift, and retention time consistency: [evidence span withheld — anchor status: not_found]
- [methods] M4 metric set from Zhang et al. (2014) — sharpness, Gaussian similarity, triangle peak area similarity ratio (TPASR), and local zigzag index: [evidence span withheld — anchor status: not_found]
