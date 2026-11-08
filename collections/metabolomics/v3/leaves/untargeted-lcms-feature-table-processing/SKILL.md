---
name: untargeted-lcms-feature-table-processing
description: Use when an untargeted LC–MS dataset has already been peak-picked, retention-time
  corrected, and grouped in XCMS, and you need chromatographic quality features (sharpness,
  jaggedness, symmetry, modality, etc.) per EIC as classifier input.
when_to_use_negative:
- Input is a uniquely formatted vendor-processed output rather than an XCMS object
  — MetaClean requires an XCMS object input and is not amenable to vendor formats
  in its current form
- The goal is filtering by reproducibility only (e.g. RSD < 30% on pooled QCs) without
  chromatographic shape assessment — plain RSD filtering is the simpler alternative
  the authors compared against
- Peaks have not yet been pre-processed (peak-picked, RT-corrected, grouped) in XCMS
  — the skill expects a filled feature table and EICs, not raw LC–MS files
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: MetaClean
  role: R package providing the 12 peak-quality metric functions (calculateApexMaxBoundaryRatio,
    calculateJaggedness, calculateSymmetry, calculateModality, calculateFWHM, calculateElutionShift,
    calculateRetentionTimeConsistency, calculateSharpness, calculateGaussianSimilarity,
    calculateTPASR, calculateZigZagIndex) and getEvalObj() used to build the metric
    feature sets
  repo: https://CRAN.R-project.org/package=MetaClean
- name: XCMS
  role: 'Upstream pre-processing: peak-picking, retention time correction, grouping
    across samples and QCs, and generation of the xcmsEIC/filled peak objects that
    MetaClean consumes'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:1808565fb89f215ef42b73343c0c4926b3f940192e76822ee73cf317a706a726
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaClean
  - XCMS
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# untargeted-lcms-feature-table-processing

## Summary

Compute per-peak (EIC) quality metrics from an XCMS-preprocessed untargeted LC–MS dataset using MetaClean's 12 peak-quality metrics, organized into the M4 (Zhang et al. 2014), M7 (TargetedMSQC-derived), and combined M11 feature sets for downstream Pass/Fail classifier training.

## When to use

Use when an untargeted LC–MS dataset has already been peak-picked, retention-time corrected, and grouped in XCMS, and you need chromatographic quality features (sharpness, jaggedness, symmetry, modality, etc.) per EIC as classifier input.

## When NOT to use

- Input is a uniquely formatted vendor-processed output rather than an XCMS object — MetaClean requires an XCMS object input and is not amenable to vendor formats in its current form
- The goal is filtering by reproducibility only (e.g. RSD < 30% on pooled QCs) without chromatographic shape assessment — plain RSD filtering is the simpler alternative the authors compared against
- Peaks have not yet been pre-processed (peak-picked, RT-corrected, grouped) in XCMS — the skill expects a filled feature table and EICs, not raw LC–MS files

## Inputs

- XCMS pre-processed dataset: xcmsEIC object from getEIC() plus filled peak object from fillPeaks()
- flatness.factor noise-threshold parameter for the metric calculations
- Optional expert Pass/Fail labels (high-quality vs low-quality peaks) appended as columns

## Outputs

- evalObj containing per-EIC chromatographic data ready for MetaClean metric functions
- Per-peak metric matrices of dimension 500 x M (M = 4, 7, 11 for M4, M7, M11 sets)
- metric_tables.csv — one table per metric set with EIC numbers and optional Pass/Fail labels as columns

## How to apply

Start from the XCMS pre-processing outputs: run peak-picking, retention time correction, and grouping of peaks across samples and QCs in XCMS, then generate filled peak data and extract chromatograms into an xcmsEIC object via getEIC(). Convert the chromatographic data into a MetaClean evalObj with getEvalObj(xs, fill). Compute the M7 set adapted from TargetedMSQC (apex-boundary ratio, jaggedness, symmetry, modality, FWHM2base, elution shift, retention time consistency) with the flatness.factor noise threshold, and the M4 set from Zhang et al. (2014) (sharpness, Gaussian similarity, triangle peak area similarity ratio, local zigzag index). For each EIC, compute every metric across all individual samples and take the mean per sample to yield one per-peak quality value, assembling a 500 x M matrix (M = 4, 7, or 11). Write one metric table per feature set — with EIC numbers and optional Pass/Fail labels as columns — to metric_tables.csv for classifier training and testing.

## Related tools

- **MetaClean** (R package providing the 12 peak-quality metric functions (calculateApexMaxBoundaryRatio, calculateJaggedness, calculateSymmetry, calculateModality, calculateFWHM, calculateElutionShift, calculateRetentionTimeConsistency, calculateSharpness, calculateGaussianSimilarity, calculateTPASR, calculateZigZagIndex) and getEvalObj() used to build the metric feature sets) — https://CRAN.R-project.org/package=MetaClean
- **XCMS** (Upstream pre-processing: peak-picking, retention time correction, grouping across samples and QCs, and generation of the xcmsEIC/filled peak objects that MetaClean consumes)

## Examples

```
remotes::install_github("KelseyChetnik/MetaClean"); evalObj <- getEvalObj(xs, fill)
```

## Evaluation signals

- The assembled metric matrix has exactly M = 4, 7, or 11 metric columns for the M4, M7, and M11 sets respectively, one row per EIC (e.g. 500 rows)
- Each metric value in the table is a per-peak mean computed across all N individual samples, not a single-sample value
- The M7 metrics reproduce the TargetedMSQC definitions (apex-boundary ratio, jaggedness, symmetry, modality, FWHM2base, elution shift, retention time consistency) applied with the flatness.factor noise threshold
- Output CSV contains EIC numbers as row identifiers and, when labels are supplied, Pass/Fail columns suitable for classifier training/testing input
- Downstream cross-validation (fivefold CV over 3 metric sets x 8 algorithms = 24 classifiers) can be run directly on the metric tables

## Limitations

- MetaClean requires an XCMS object input and is not amenable in its current form to uniquely formatted vendor-processed outputs
- The classifier was conservative and performed worse for the low-quality (Fail) class than the high-quality one, partly because the development dataset contained fewer low-quality peaks
- Low-abundance peaks are difficult to distinguish from background noise; the development dataset did not contain many low-abundance Pass peak examples
- Ionization mode, analytical platform and method, and biological matrix affect peak shape and size, and thus metric behavior — retraining/splitting by mode (e.g. MTBLS306 positive vs negative) may be needed

## Evidence
