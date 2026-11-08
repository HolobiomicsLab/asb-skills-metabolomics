---
name: mass-chromatogram-noise-similarity-indexing
description: Use when filtering raw EIC intensity profiles from LC/MS-based metabolomics
  data for spike/background noise prior to chromatographic peak detection, or when
  benchmarking the MCQ index against alternative EIC quality metrics such as the EIC-related
  global zigzag index.
when_to_use_negative:
- The EICs being evaluated have high background signals or contain only a single peak
  — the paper explicitly states the MCQ index cannot fairly evaluate these cases;
  use the EIC-related global zigzag index instead
- The task is evaluating detected chromatographic peaks (not raw EICs) — the paper's
  peak-level metrics (sharpness, Gaussian similarity, SNR, peak significance level,
  TPASR, local peak-related zigzag index) apply at that downstream stage
- You need a reproducible end-to-end pipeline out of the box — the LC/MS profile dataset
  and Matlab scripts are available only upon request and no license is specified
tools:
- name: Matlab
  role: Implementation language for the four-module data processing program in which
    the MCQ index (similarity to smoothed and mean-subtracted chromatogram versions)
    is computed during the EIC quality evaluation and filtering module
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:ad1ff9d7a0fec3e9d712974328e639c1f3d99354382b0e032bd435ab3ef4b58e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Matlab
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# mass-chromatogram-noise-similarity-indexing

## Summary

Computes the mass chromatographic quality (MCQ) index for extracted ion chromatograms (EICs) by measuring the similarity of each original mass chromatogram to its smoothed version (spike detection) and its mean-subtracted version (background detection), as described by Zhang & Zhao (2014), to flag noisy EICs before downstream peak detection.

## When to use

Use when filtering raw EIC intensity profiles from LC/MS-based metabolomics data for spike/background noise prior to chromatographic peak detection, or when benchmarking the MCQ index against alternative EIC quality metrics such as the EIC-related global zigzag index.

## When NOT to use

- The EICs being evaluated have high background signals or contain only a single peak — the paper explicitly states the MCQ index cannot fairly evaluate these cases; use the EIC-related global zigzag index instead
- The task is evaluating detected chromatographic peaks (not raw EICs) — the paper's peak-level metrics (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local peak-related zigzag index) apply at that downstream stage
- You need a reproducible end-to-end pipeline out of the box — the LC/MS profile dataset and Matlab scripts are available only upon request and no license is specified

## Inputs

- EIC intensity profiles extracted from an LC/MS profile dataset (e.g., representative EIC_IDs 27, 40, 90, 134, 154, 167 among 611 extracted EICs)
- LC/MS profile dataset acquired on an Agilent HPLC system interfaced with a Q-TOF Premier mass spectrometer

## Outputs

- Per-EIC MCQ index values (similarity to smoothed version for spikes, to mean-subtracted version for background)
- Per-EIC global zigzag index values in [0, 4.0] (for the paired robustness comparison)
- Tab-separated table of per-EIC metric values with qualitative good/bad verdicts under each metric

## How to apply

For each EIC intensity profile, implement the MCQ index in Matlab by computing the similarity of the original mass chromatogram to its smoothed version (spike detection) and to its mean-subtracted version (background detection), as defined in the paper's Methods; low similarity signals spike or background contamination. Position this step as module 2 of the four-module Matlab pipeline: 1) acute EIC extraction, 2) EIC quality evaluation and filtering, 3) chromatographic peak detection, 4) peak quality evaluation and filtering — i.e., filtering happens before the time-consuming peak-detection stage. Critically, interpret MCQ with the documented caveat that it cannot fairly evaluate EICs with high background signals or those containing only a single peak; for those scenarios pair it with the EIC-related global zigzag index computed from first-order derivatives — sum of (2*In − In−1 − In+1)^2 over neighboring points, normalized by N times the squared effective peak intensity (max intensity minus baseline at apex), yielding values in [0, 4.0]. On representative EICs (e.g., EIC_IDs 27, 40, 90, 134, 154, 167 out of 611 extracted EICs), expect the failure pattern where a locally 'bad' EIC (167) gets a low MCQ and high zigzag, while high-background or single-peak EICs (27/90/134/154) get low MCQ values yet low zigzag values. Write per-EIC metric values and good/bad verdicts to a tab-separated table for inspection.

## Related tools

- **Matlab** (Implementation language for the four-module data processing program in which the MCQ index (similarity to smoothed and mean-subtracted chromatogram versions) is computed during the EIC quality evaluation and filtering module)

## Evaluation signals

- On representative EICs, a known-bad EIC (EIC_ID 167) receives a low MCQ and a high zigzag index, while high-background or single-peak EICs (EIC_IDs 27/90/134/154) receive low MCQ values yet low zigzag values — reproducing the paper's reported pattern
- All computed global zigzag index values fall within the documented range [0, 4.0]
- The output table contains both a numeric metric value and a qualitative good/bad verdict under each metric for every evaluated EIC
- The MCQ computation is applied at pipeline module 2 (EIC quality evaluation and filtering), before chromatographic peak detection — filtering must precede peak detection
- The two MCQ components (spike similarity and background similarity) are computed against the smoothed version and the mean-subtracted version of the same chromatogram, respectively

## Limitations

- The MCQ index cannot fairly evaluate EICs with high background signals or those containing only a single peak; it is suited for selecting and aligning analyte components but not for fair quality scoring of these EIC types
- Setting an optimal cutoff threshold for each quality metric remains an unsolved challenge
- The LC/MS profile dataset and Matlab scripts are available only upon request; no public repository and no license for usage are provided
- Machine learning-based automatic peak identification (e.g., SVM) was not implemented because a curated training set of good and bad representative peaks does not yet exist

## Evidence

- [abstract] Although the MCQ index is suited for selecting and aligning analyte components, it cannot fairly evaluate EICs with high background signals or those containing only a single peak.: "Although the MCQ index is suited for selecting and aligning analyte components, it cannot fairly evaluate EICs with high background signals or those containing only a single peak."
- [abstract] Our proposed EIC related global zigzag index is robust enough to evaluate EIC qualities in both scenarios.: "Our proposed EIC related global zigzag index is robust enough to evaluate EIC qualities in both scenarios."
- [methods] we developed our own data processing program in Matlab, which consists of four sequential modules: "we developed our own data processing program in Matlab, which consists of four sequential modules"
- [methods] 2) EIC quality evaluation and filtering, 3) chromatographic peak detection, and 4) peak quality evaluation and filtering: "2) EIC quality evaluation and filtering, 3) chromatographic peak detection, and 4) peak quality evaluation and filtering"
- [discussion] setting an optimal cutoff threshold for each metric still is a challenge task: "setting an optimal cutoff threshold for each metric still is a challenge task"
- [discussion] The LC/MS profile dataset and Matlab scripts are available upon request.: "The LC/MS profile dataset and Matlab scripts are available upon request."
