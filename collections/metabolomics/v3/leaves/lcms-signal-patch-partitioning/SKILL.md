---
name: lcms-signal-patch-partitioning
description: Use when raw LC/MS data in mzML format must be converted to an MS1 ion-feature
  table (median rt, m/z, normalized intensities) and no QC samples or user-tuned peak-detection
  thresholds are available.
when_to_use_negative:
- The input is already a feature/peak table (e.g., the mzLearn 2,736-feature CSV matrix)
  rather than raw mzML files — this skill operates on raw spectral data
- De-isotoping, compound annotation, or MS/MS identification is the goal — mzLearn
  results are at the ion MS1 level only and those steps require downstream workflows
- Detection of MS2-level features or integration of MS/MS spectra into the detection
  step is required
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3344
- http://edamontology.org/topic_0092
tools:
- name: mzLearn
  role: Implements the patch-partitioning, auto/cross-correlation and pin-threshold
    learning core that runs from mzML files without user-set parameters
  repo: github.com/ReviveMed/mzEmbed
- name: XCMS
  role: Baseline comparator for signal-detection output (7,100 signals on average
    vs mzLearn's 11,442 across 15 public datasets)
- name: ASARI
  role: Baseline comparator for signal-detection output (4,655 signals on average
    across 15 public datasets)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:d11b318ed1b2c4a1f4a5e30e699ace139c261ce2b424461fa98339c28eee8324
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  tools:
  - ASARI
  - XCMS
  - mzLearn
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3344
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# lcms-signal-patch-partitioning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A data-driven MS1 signal-detection strategy from mzLearn that recursively partitions the retention-time/m/z space into dynamically resized patches, filters them by auto-correlation, and iteratively learns thresholds from high-quality 'pin' patches without user-set parameters.

## When to use

Use when raw LC/MS data in mzML format must be converted to an MS1 ion-feature table (median rt, m/z, normalized intensities) and no QC samples or user-tuned peak-detection thresholds are available.

## When NOT to use

- The input is already a feature/peak table (e.g., the mzLearn 2,736-feature CSV matrix) rather than raw mzML files — this skill operates on raw spectral data
- De-isotoping, compound annotation, or MS/MS identification is the goal — mzLearn results are at the ion MS1 level only and those steps require downstream workflows
- Detection of MS2-level features or integration of MS/MS spectra into the detection step is required

## Inputs

- Raw LC/MS data files in open-source mzML format (mzLearn accepts batches such as 2,075 files in a single run)

## Outputs

- Two-dimensional table of detected MS1 ion features defined by median rt and m/z values, with normalized intensities across samples
- Retention-time and intensity drift maps learned iteratively from the data

## How to apply

Load the MS1 spectral data from the input mzML files, then recursively partition the retention-time and m/z space into patches that are dynamically resized according to local signal density. Apply auto-correlation filtering within each patch to separate true chromatographic peaks from noise, estimating correlation thresholds from the highest-quality 'pin' patches via an iterative loop that converges after roughly three iterations. Perform sliding cross-correlation alignment across files to build the rt/intensity drift map (generated alongside synthetic QC samples) so signals are matched between samples despite batch effects and run-order drift. Assemble the filtered, aligned signals into a final two-dimensional feature table of median rt and m/z values with normalized per-sample intensities. Rationale: being deterministic and non-parametric, the iterative learning avoids hand-tuned thresholds while correcting drift without experimental QC samples.

## Related tools

- **mzLearn** (Implements the patch-partitioning, auto/cross-correlation and pin-threshold learning core that runs from mzML files without user-set parameters) — github.com/ReviveMed/mzEmbed
- **XCMS** (Baseline comparator for signal-detection output (7,100 signals on average vs mzLearn's 11,442 across 15 public datasets))
- **ASARI** (Baseline comparator for signal-detection output (4,655 signals on average across 15 public datasets))

## Evaluation signals

- Per-dataset signal counts should be consistent with the reported behavior (11,442 signals on average across 15 public datasets; >60% more signals than XCMS/ASARI on average)
- True positive rate should be at or above the reported 89.0%, with a false positive rate near or below 12.5%, versus XCMS (77.4% TP / 17.3% FP) and ASARI (49.6% TP / 18.8% FP)
- The iterative threshold learning should converge (about three iterations) and yield a final feature list with minimal missing values aligned across samples
- Output must be a 2D feature table keyed by median rt and m/z with normalized intensities — not raw patch or correlation artifacts

## Limitations

- Outputs are limited to the ion MS1 level; mzLearn does not perform de-isotoping or MS/MS identification
- Iterative learning is computationally heavy — analysis can take up to 2.5 days on a single machine; parallel processing on multiple compute nodes is not yet implemented
- Public metabolomics repositories often lack standardized QC samples to estimate instrument drift, which the algorithm must infer without them

## Evidence

- [readme] README: zero-parameter iterative drift correction: [evidence span withheld — anchor status: external_doc]
