---
name: targetedmsqc-metric-adaptation
description: Use when XCMS-pre-processed untargeted LC–MS data needs per-peak quality
  features from apex-boundary ratio, jaggedness, symmetry, modality, FWHM2base, elution
  shift, and retention time consistency.
when_to_use_negative:
- The input is not an XCMS object (e.g. uniquely formatted vendor-processed outputs
  from MassHunter Profinder, Progenesis QI, or Compound Discoverer), since MetaClean
  requires an XCMS object input
- The analysis targets targeted MS data with reference standards, where the original
  TargetedMSQC package (not its repurposing for untargeted data) is appropriate
- Only a reproducibility-based filter is needed (RSD < 30% on pooled QC samples) and
  no peak-shape quality classification is desired
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaClean
  role: Provides the metric functions (calculateApexMaxBoundaryRatio, calculateJaggedness,
    calculateSymmetry, calculateModality, calculateFWHM, calculateElutionShift, calculateRetentionTimeConsistency)
    and getEvalObj for computing the M7 set from XCMS EICs
  repo: https://CRAN.R-project.org/package=MetaClean
- name: XCMS
  role: Pre-processing and extraction of EICs (getEIC, fillPeaks) whose chromatographic
    data feed the metric computations
- name: TargetedMSQC
  role: Source of the seven original quality metrics that MetaClean repurposes for
    untargeted metabolomics
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:1807d28adce602003d9a5bfc4142b2554900ab66b06622fa107fcd589458d3a4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaClean
  - TargetedMSQC
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# targetedmsqc-metric-adaptation

## Summary

Repurposes the seven chromatographic quality metrics from TargetedMSQC (Eshghi et al. 2018) — designed for targeted MS — to untargeted LC–MS metabolomics by computing them per EIC from XCMS-extracted chromatograms, forming the M7 metric set used as classifier features in MetaClean.

## When to use

Use when XCMS-pre-processed untargeted LC–MS data needs per-peak quality features from apex-boundary ratio, jaggedness, symmetry, modality, FWHM2base, elution shift, and retention time consistency.

## When NOT to use

- The input is not an XCMS object (e.g. uniquely formatted vendor-processed outputs from MassHunter Profinder, Progenesis QI, or Compound Discoverer), since MetaClean requires an XCMS object input
- The analysis targets targeted MS data with reference standards, where the original TargetedMSQC package (not its repurposing for untargeted data) is appropriate
- Only a reproducibility-based filter is needed (RSD < 30% on pooled QC samples) and no peak-shape quality classification is desired

## Inputs

- xcmsEIC object from XCMS getEIC() with the filled peak table from fillPeaks()
- flatness.factor noise-threshold parameter used by the MetaClean metric functions
- Optionally, expert Pass/Fail labels per EIC to attach as classifier-label columns

## Outputs

- Per-peak values for the 7 TargetedMSQC-derived metrics (M7 set), averaged across samples
- Metric table with EIC numbers as rows, M7 metric values (and optional Pass/Fail labels) as columns, e.g. metric_tables.csv, usable as a P x M classifier feature matrix

## How to apply

Load the XCMS-pre-processed dataset as an xcmsEIC object (from getEIC() and the filled peak table from fillPeaks()) and convert it to an evalObj via getEvalObj(xs, fill). Then compute the M7 metric set — apex-boundary ratio, jaggedness, symmetry, modality, FWHM2base, elution shift, and retention time consistency — with MetaClean functions calculateApexMaxBoundaryRatio, calculateJaggedness, calculateSymmetry, calculateModality, calculateFWHM, calculateElutionShift, and calculateRetentionTimeConsistency, applying the flatness.factor noise threshold where relevant. Each metric is computed across all N individual samples for a selected peak and averaged per sample to yield one per-peak value, so a panel of P peaks yields a P x 7 feature matrix. The rationale is that these targeted-data metrics capture integration quality (shape, jaggedness, boundary placement, elution alignment) even when no reference standard exists; the resulting M7 columns are combined with Zhang et al. (2014) metrics to form the M11 set, which paired with AdaBoost was the best-performing MetaClean classifier. Write the metric table (EIC numbers as rows, metric values and optional Pass/Fail labels as columns) to CSV for classifier training/testing.

## Related tools

- **MetaClean** (Provides the metric functions (calculateApexMaxBoundaryRatio, calculateJaggedness, calculateSymmetry, calculateModality, calculateFWHM, calculateElutionShift, calculateRetentionTimeConsistency) and getEvalObj for computing the M7 set from XCMS EICs) — https://CRAN.R-project.org/package=MetaClean
- **XCMS** (Pre-processing and extraction of EICs (getEIC, fillPeaks) whose chromatographic data feed the metric computations)
- **TargetedMSQC** (Source of the seven original quality metrics that MetaClean repurposes for untargeted metabolomics)

## Examples

```
evalObj <- getEvalObj(xs, fill = TRUE); ratio <- calculateApexMaxBoundaryRatio(evalObj)
```

## Evaluation signals

- The evalObj is successfully constructed from the xcmsEIC object and the filled peak table without errors
- Each of the 7 M7 metrics yields a finite, per-peak value averaged across all N individual samples
- The resulting feature matrix has one column per M7 metric (7 columns for M7, or 11 when combined into M11) with rows matching the selected EIC numbers
- Metric table columns align with the paper's M7 metric names and are formatted with EIC numbers and optional Pass/Fail labels for classifier input
- Downstream, M7-based classifiers (Random Forest was best for M7) perform comparably to the paper's cross-validation framework

## Limitations

- Metrics are computed on chromatograms from XCMS objects, so uniquely formatted vendor-processed outputs are not amenable
- The classifier trained on these metrics performed worse for the low-quality (Fail) class than the high-quality one, partly because development sets contained fewer low-quality peaks
- Peak shape and size depend on ionization mode, analytical platform and method, and biological matrix, so metric behavior may shift across datasets (e.g. Test 2 was more dissimilar from the development set)

## Evidence
