---
name: lc-ms-feature-quality-metric-computation
description: Use when you have an XCMS peak-picked LC–MS untargeted metabolomics run
  (with pooled QC samples) and need numeric per-peak quality metrics to feed peak-quality
  classifiers or flag low-quality integrations.
when_to_use_negative:
- You are already working with vendor-processed outputs that MetaClean cannot accept
  — the article notes MetaClean requires an XCMS object input and is not amenable
  to uniquely formatted vendor-processed outputs
- You only need binary Pass/Fail flags and have no interest in the underlying chromatographic
  quality metrics — call the classifier wrapper instead
- Your data is targeted MS/MS with no pooled QC structure — the metric definitions
  assume untargeted LC–MS with dilution and replicate relationships
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: XCMS
  role: Upstream pre-processing that produces the peak table and grouped chromatographic
    peaks the quality metrics are computed on
- name: MetaClean
  role: R package exposing the peak-quality metric calculation functions used to build
    the M11 feature matrix
  repo: https://CRAN.R-project.org/package=MetaClean
- name: IPO
  role: Optimizes XCMS parameters before metric computation so peaks are integrated
    consistently across runs
- name: TargetedMSQC
  role: Source of seven of the chromatographic shape metrics adapted for untargeted
    metabolomics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-020-01738-3
    title: 'MetaClean: a machine learning-based classifier for reduced false positive
      peak detection in untargeted LC–MS metabolomics data'
  source_content_hash: sha256:2a3a0bff2d66c2b4ab7003737dfcc53003cc0884988a770c97495791f7fa4276
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - IPO
  - MetaClean
  - TargetedMSQC
  - XCMS
  - fastAdaboost
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# LC-MS feature quality metric computation

## Summary

Computes 12 peak-quality metrics (11 MetaClean metrics from Zhang et al. 2014 and TargetedMSQC-derived sources, plus additional features) on an XCMS-derived feature table so downstream classifiers like AdaBoost+M11 can binarize peaks into Pass/Fail. This is the upstream step that turns raw LC–MS integration quality into numeric feature vectors.

## When to use

Use when you have an XCMS peak-picked LC–MS untargeted metabolomics run (with pooled QC samples) and need numeric per-peak quality metrics to feed peak-quality classifiers or flag low-quality integrations.

## When NOT to use

- You are already working with vendor-processed outputs that MetaClean cannot accept — the article notes MetaClean requires an XCMS object input and is not amenable to uniquely formatted vendor-processed outputs
- You only need binary Pass/Fail flags and have no interest in the underlying chromatographic quality metrics — call the classifier wrapper instead
- Your data is targeted MS/MS with no pooled QC structure — the metric definitions assume untargeted LC–MS with dilution and replicate relationships

## Inputs

- XCMS object with peak-picked, retention-time-corrected and grouped features (chromatographic peak table and sample/QC metadata)
- Raw LC–MS metabolomics run including biological samples, dilution series and technical replicates
- Expert Pass/Fail labels on a subset of peaks, when the goal is to train rather than just score a classifier

## Outputs

- Per-peak numeric metric matrix (one row per XCMS feature, one column per quality metric such as medial sharpest rise ratio, medial sharpness ratio, medial Gaussian similarity, correlation with dilution, correlation across technical replicates, elution peak tailing/symmetry/width, m/z scatter)
- Feature table annotated with Pass/Fail labels after classification
- The 11-metric (M11) representation used to train and evaluate the AdaBoost global peak-quality classifier

## How to apply

Start from XCMS output where peak-picking, retention time correction, and grouping across samples and QCs have been performed (the article recommends IPO-optimized parameters). Compute per-peak quality metrics via the MetaClean R package: seven metrics repurposed from TargetedMSQC (e.g. medial sharpest rise ratio, medial sharpness ratio, medial Gaussian similarity, correlation with dilution and correlation across technical replicates) plus four Zhang et al. metrics (elution peak tailing, elution peak symmetry, elution peak width, m/z scatter). Take the union of these metric sets as the M11 feature vector for the peaks in every sample (biological samples and pooled QCs), producing a metrics matrix aligned to the XCMS feature table row order. The rationale is that integration quality correlates with these chromatographic shapes, so a supervised classifier can separate Pass from Fail peaks without per-peak manual inspection. Keep the pooled-QC column structure intact because the metrics for correlation across dilutions and technical replicates depend on the dilution/replicate relationships encoded in sample metadata.

## Related tools

- **XCMS** (Upstream pre-processing that produces the peak table and grouped chromatographic peaks the quality metrics are computed on)
- **MetaClean** (R package exposing the peak-quality metric calculation functions used to build the M11 feature matrix) — https://CRAN.R-project.org/package=MetaClean
- **IPO** (Optimizes XCMS parameters before metric computation so peaks are integrated consistently across runs)
- **TargetedMSQC** (Source of seven of the chromatographic shape metrics adapted for untargeted metabolomics)

## Examples

```
R> library(MetaClean); metrics <- cleanMaybeBad(data = xcmsObject, qc = TRUE, qcTag = "QC")
```

## Evaluation signals

- The metric matrix has one row per XCMS peak and no missing values for peaks with complete chromatographic data
- Metric distributions differ qualitatively between expert-labeled Pass and Fail peaks (Fail peaks tend to have worse medial sharpness / Gaussian similarity)
- The 11-metric (M11) representation, when fed to AdaBoost and evaluated on Test 1, reproduces the reported accuracy and Pass/Fail F-measure behavior described for the same-platform validation
- Metric columns match the names/definitions used by the MetaClean package's exported functions (12 metrics as listed in the README)
- Recomputing metrics on the same XCMS object gives identical values (deterministic, no randomness)

## Limitations

- Metric computation presupposes XCMS pre-processing — vendor-supplied feature tables are not accepted
- Peak quality depends on ionization mode, analytical platform, and biological matrix, so metrics and classifier performance do not transfer blindly across platforms
- Manual quality assessment remains the ground-truth standard; automated metric-based flagging alone is not sufficient to guarantee no poor integrations propagate downstream
- Seven of the eleven metrics derive from TargetedMSQC, which was designed for targeted MS, so their repurposing for untargeted data is an adaptation, not a native design

## Evidence
