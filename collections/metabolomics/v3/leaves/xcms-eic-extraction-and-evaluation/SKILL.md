---
name: xcms-eic-extraction-and-evaluation
description: Use when an XCMS-processed untargeted LC–MS dataset (getEIC()/fillPeaks()
  output) needs per-peak quality features, or when Pass/Fail peak-quality labels must
  be derived before ML-based peak filtering.
when_to_use_negative:
- Input is a vendor-processed output in a uniquely formatted vendor format — MetaClean
  requires an XCMS object input and is not amenable to such outputs
- The dataset has not been pre-processed with XCMS (peak-picking, RT correction, grouping),
  since no xcmsEIC/fillPeaks objects exist to extract chromatograms from
- Only relative standard deviation filtering on pooled QCs is needed (RSD < 30% threshold)
  rather than shape-based peak-quality classification
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: XCMS
  role: Pre-processing software whose getEIC() and fillPeaks() outputs supply the
    xcmsEIC and filled objects from which chromatographic data are extracted into
    the evalObj
- name: MetaClean
  role: R package providing getEvalObj() and the calculate* metric functions (calculateApexMaxBoundaryRatio,
    calculateJaggedness, calculateSymmetry, calculateModality, calculateFWHM, calculateElutionShift,
    calculateRetentionTimeConsistency, calculateSharpness, calculateGaussianSimilarity,
    calculateTPASR, calculateZigZagIndex) used to build the M4/M7/M11 feature sets
  repo: https://CRAN.R-project.org/package=MetaClean
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:44407993c46b3eddde5da2dbc5c91026a905b719930c669ff8c7b7f69c4dfd57
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MetaClean
  - XCMS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# xcms-eic-extraction-and-evaluation

## Summary

Extracts chromatographic EIC data from an XCMS pre-processed untargeted LC–MS dataset into a MetaClean evalObj and computes the 12 MetaClean peak-quality metrics (M4, M7, M11 feature sets) per EIC as classifier features.

## When to use

Use when an XCMS-processed untargeted LC–MS dataset (getEIC()/fillPeaks() output) needs per-peak quality features, or when Pass/Fail peak-quality labels must be derived before ML-based peak filtering.

## When NOT to use

- Input is a vendor-processed output in a uniquely formatted vendor format — MetaClean requires an XCMS object input and is not amenable to such outputs
- The dataset has not been pre-processed with XCMS (peak-picking, RT correction, grouping), since no xcmsEIC/fillPeaks objects exist to extract chromatograms from
- Only relative standard deviation filtering on pooled QCs is needed (RSD < 30% threshold) rather than shape-based peak-quality classification

## Inputs

- XCMS pre-processed untargeted LC–MS dataset (xcmsEIC object from getEIC() plus the filled object from fillPeaks())
- flatness.factor noise-threshold parameter for MetaClean metric computation
- Optional expert Pass/Fail labels per peak for training-set construction

## Outputs

- MetaClean evalObj holding per-sample chromatographic data for each EIC
- Peak-quality metric tables per feature set (M4, M7, M11) as a 500 x M matrix with EIC numbers and optional Pass/Fail columns
- metric_tables.csv for classifier training/testing input

## How to apply

Start from the XCMS pre-processed dataset: run peak-picking, retention time correction, and grouping across samples and QCs with XCMS, then load the xcmsEIC object from getEIC() together with the filled peak object from fillPeaks() and extract chromatographic data into an evalObj via getEvalObj(xs, fill). Compute the M7 metric set adapted from TargetedMSQC (apex-boundary ratio, jaggedness, symmetry, modality, FWHM2base, elution shift, retention time consistency) using MetaClean functions such as calculateApexMaxBoundaryRatio, calculateJaggedness, calculateSymmetry, calculateModality, calculateFWHM, calculateElutionShift, and calculateRetentionTimeConsistency with the flatness.factor noise threshold. Compute the M4 set from Zhang et al. (2014) — sharpness, Gaussian similarity, triangle peak area similarity ratio (TPASR), and local zigzag index — via calculateSharpness, calculateGaussianSimilarity, calculateTPASR, and calculateZigZagIndex. For each selected peak, average each metric across all N individual samples to a per-peak value and assemble a 500 x M matrix (M = 4, 7, 11 for M4, M7, M11). Write one metric table per set, with EIC numbers and optional Pass/Fail labels as columns, to metric_tables.csv as classifier training/testing input.

## Related tools

- **XCMS** (Pre-processing software whose getEIC() and fillPeaks() outputs supply the xcmsEIC and filled objects from which chromatographic data are extracted into the evalObj)
- **MetaClean** (R package providing getEvalObj() and the calculate* metric functions (calculateApexMaxBoundaryRatio, calculateJaggedness, calculateSymmetry, calculateModality, calculateFWHM, calculateElutionShift, calculateRetentionTimeConsistency, calculateSharpness, calculateGaussianSimilarity, calculateTPASR, calculateZigZagIndex) used to build the M4/M7/M11 feature sets) — https://CRAN.R-project.org/package=MetaClean

## Examples

```
remotes::install_github("KelseyChetnik/MetaClean"); evalObj <- getEvalObj(xs, fill)
```

## Evaluation signals

- Metric matrix has expected shape: one row per selected peak (e.g. 500 peaks) and M = 4, 7, or 11 columns for the M4, M7, and M11 sets respectively
- Each metric value is a per-peak mean across all N individual samples, so check that averaging over samples was applied consistently for every metric
- Output tables contain EIC numbers as row identifiers and optional Pass/Fail labels as columns, matching the metric_tables.csv schema
- Verify each M7 metric (apex-boundary ratio, jaggedness, symmetry, modality, FWHM2base, elution shift, RT consistency) and M4 metric (sharpness, Gaussian similarity, TPASR, zigzag index) is present and finite per EIC
- Confirm the flatness.factor noise threshold was applied when computing noise-sensitive metrics such as jaggedness and zigzag index

## Limitations

- MetaClean requires an XCMS object input and is not amenable in its current form to uniquely formatted vendor-processed outputs
- The classifier development data contained few low-abundance Pass peak examples, so metrics may not separate low-abundance peaks from background noise well
- Ionization mode, analytical platform and method, and biological matrix affect peak shape and size and must be considered when applying the methodology

## Evidence

- [other] Load the XCMS pre-processed dataset (xcmsEIC object from getEIC() and fill object from fillPeaks()) and extract chromatographic data into an evalObj via getEvalObj(xs, fill).: [evidence span withheld — anchor status: not_found]
