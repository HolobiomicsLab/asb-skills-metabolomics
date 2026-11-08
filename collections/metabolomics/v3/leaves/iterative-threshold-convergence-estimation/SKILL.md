---
name: iterative-threshold-convergence-estimation
description: Use when processing raw LC/MS mzML files where signal-detection thresholds
  cannot be set manually and must be learned from the data itself until convergence.
when_to_use_negative:
- Input is already a processed feature table — the convergence procedure operates
  on raw mzML MS1 data, not on feature matrices (downstream mzEmbed consumes CSV feature
  matrices directly)
- You need de-isotoping or MS/MS identification — mzLearn outputs are at the ion MS1
  level only and these steps must be done with downstream workflows
- You need a fast single-pass method — the iterative learning increases computational
  time, taking up to 2.5 days on a single machine
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzLearn
  role: 'Implements the iterative, deterministic, non-parametric threshold-learning
    core: patch partitioning of rt/m/z space, auto-correlation filtering, and pin-based
    threshold estimation until convergence, run from mzML files without user-set parameters'
  repo: github.com/ReviveMed/mzEmbed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:f8b2ee9aeb8e70eeb901ac33339e92b74f720d7a4cde8371c7682fcdbba7e155
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  tools:
  - mzLearn
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# iterative-threshold-convergence-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

mzLearn's data-driven, non-parametric mechanism for estimating signal-detection thresholds and rt/intensity drift maps entirely from the LC/MS data itself: rt/m/z space is partitioned into patches, filtered by auto-correlation, and thresholds are learned iteratively from the highest-quality 'pin' patches until convergence (~3 iterations), so no user-set parameters or QC samples are required.

## When to use

Use when processing raw LC/MS mzML files where signal-detection thresholds cannot be set manually and must be learned from the data itself until convergence.

## When NOT to use

- Input is already a processed feature table — the convergence procedure operates on raw mzML MS1 data, not on feature matrices (downstream mzEmbed consumes CSV feature matrices directly)
- You need de-isotoping or MS/MS identification — mzLearn outputs are at the ion MS1 level only and these steps must be done with downstream workflows
- You need a fast single-pass method — the iterative learning increases computational time, taking up to 2.5 days on a single machine

## Inputs

- Raw LC/MS data files in the open-source mzML format (MS1 spectral data)
- Large-scale cohorts of mzML files (e.g., 2,075 files in a single run); no QC samples required

## Outputs

- A two-dimensional table of detected MS1 ion features defined by median rt and m/z values with normalized intensities across samples
- Learned internal signal-detection thresholds and intensity/retention-time drift maps
- Final high-quality feature list with minimal missing values aligned across samples (e.g., the robust 2,736-feature matrix used downstream)

## How to apply

Load raw MS1 spectral data from mzML files and recursively partition the retention-time and m/z space into patches that are dynamically resized according to local signal density. Apply auto-correlation filtering to each patch to separate true chromatographic peaks from noise, then identify the highest-quality 'pin' patches and estimate correlation thresholds from them via an iterative procedure that repeats until convergence, which occurs after roughly three iterations. Alongside threshold learning, construct synthetic QC samples and build intensity and retention-time drift maps through the same dynamic, iterative, data-driven learning process, correcting rt and intensity drifts caused by batch effects and run order. The rationale is that a deterministic, non-parametric algorithm that learns internal thresholds from the data removes the need for user-tuned parameters or experimental QC samples. The converged output is a two-dimensional feature table defined by median rt and m/z values with normalized intensities across samples.

## Related tools

- **mzLearn** (Implements the iterative, deterministic, non-parametric threshold-learning core: patch partitioning of rt/m/z space, auto-correlation filtering, and pin-based threshold estimation until convergence, run from mzML files without user-set parameters) — github.com/ReviveMed/mzEmbed

## Evaluation signals

- The iterative threshold-estimation procedure converges after approximately three iterations
- The output is a 2D feature table with median rt and m/z values and normalized intensities across samples, with minimal missing values after alignment
- Per-dataset signal counts are consistent with the algorithm's reported behavior: on average 11,442 signals across 15 public datasets (vs 7,100 for XCMS and 4,655 for ASARI)
- No user-set parameters or experimental QC samples were needed at any step — all thresholds learned internally from the data
- When scaled to 22 public studies (20,548 blood samples), the converged robust feature set contains 2,736 metabolite signals

## Limitations

- mzLearn outputs are limited to the ion MS1 level; it does not perform de-isotoping or MS/MS identification
- Iterative learning increases computational time (up to 2.5 days on a single machine); parallel processing on multiple compute nodes is not yet implemented
- The threshold-learning pipeline was validated for, and the downstream pre-trained model developed specifically for, HILIC positive mode; other LC modes and modalities are not covered
- Public metabolomics repositories lack standardized QC samples to estimate instrument drift, which is why the method synthesizes its own QC samples

## Evidence

- [readme] iteratively learning signal characteristics: [evidence span withheld — anchor status: external_doc]
- [readme] correcting rt and intensity drifts: [evidence span withheld — anchor status: external_doc]
