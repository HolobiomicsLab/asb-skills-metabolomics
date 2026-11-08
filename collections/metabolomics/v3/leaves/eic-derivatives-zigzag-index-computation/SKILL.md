---
name: eic-derivatives-zigzag-index-computation
description: Use when scoring/filtering EICs extracted from LC/MS metabolomics profiles
  prior to peak detection, especially for EICs with high background signal or only
  a single peak where the MCQ index is unfair.
when_to_use_negative:
- Scoring already-detected chromatographic peaks — the paper's six peak-level metrics
  (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local peak-related
  zigzag index) are the appropriate tools there, not the EIC-level global zigzag index
- Selecting or aligning analyte components, where the MCQ index is the suited metric
- Input is a peak list or feature table with no per-point EIC intensity vector, since
  the index requires the full first-order derivative structure of the chromatogram
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: Matlab
  role: Language of the paper's own four-module data processing program in which the
    EIC quality evaluation and filtering (zigzag index) module is implemented
- name: Python
  role: Language for a self-contained, deterministic reimplementation of the global
    zigzag index and synthetic EIC validation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:61e096d7c3b03b2d844fba7a8f4cd86052486257a25d517b9eea1e4c4a6d203f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Matlab
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# eic-derivatives-zigzag-index-computation

## Summary

Computes the EIC-related global zigzag index, a quality metric for extracted ion chromatograms (EICs) in LC/MS-based metabolomics, derived from an EIC's first-order derivatives. It robustly scores EICs that the MCQ index cannot fairly evaluate, i.e. those with high background signal or containing only a single peak.

## When to use

Use when scoring/filtering EICs extracted from LC/MS metabolomics profiles prior to peak detection, especially for EICs with high background signal or only a single peak where the MCQ index is unfair.

## When NOT to use

- Scoring already-detected chromatographic peaks — the paper's six peak-level metrics (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local peak-related zigzag index) are the appropriate tools there, not the EIC-level global zigzag index
- Selecting or aligning analyte components, where the MCQ index is the suited metric
- Input is a peak list or feature table with no per-point EIC intensity vector, since the index requires the full first-order derivative structure of the chromatogram

## Inputs

- Extracted EIC intensity array (1D vector of intensities across scan/retention-time points from an LC/MS profile dataset)
- Optional: raw LC/MS profile dataset (e.g. from Agilent HPLC + Q-TOF Premier) processed through EIC extraction to produce EIC arrays

## Outputs

- Scalar zigzag index value per EIC, within the dynamic range 0–4.0
- Good/bad EIC classification for downstream filtering (module 2 of the four-module pipeline)
- Validation report of index values on synthetic normal, high-background, and single-peak EICs

## How to apply

For each extracted EIC intensity vector I with N sampling points, compute the effective peak intensity (EPI = max intensity minus baseline at apex), first-order increments d_n = I_n - I_{n-1}, and the global zigzag index zigzag_index = Σ(2I_n - I_{n-1} - I_{n+1})^2 / (N * EPI^2), which is valid over the dynamic range 0–4.0. Implement this as a deterministic function (the paper's own program was developed in Matlab; Python works equally well) and run it as step 2 'EIC quality evaluation and filtering' of the four-module pipeline (acute EIC extraction → EIC quality evaluation and filtering → chromatographic peak detection → peak quality evaluation and filtering), so 'bad' EICs are filtered before the time-consuming peak detection step. Validate with synthetic EICs generated from a fixed random seed with documented peak width, height, and background/noise parameters: a normal EIC, a high-background EIC, and a single-peak EIC should all yield indices within 0–4.0, with the high-background and single-peak cases yielding low (good-quality) values as the paper reports. Because no optimal cutoff threshold is established, treat the index as one input to a combined-metric filter rather than a standalone hard threshold.

## Related tools

- **Matlab** (Language of the paper's own four-module data processing program in which the EIC quality evaluation and filtering (zigzag index) module is implemented)
- **Python** (Language for a self-contained, deterministic reimplementation of the global zigzag index and synthetic EIC validation)

## Examples

```
import numpy as np; z = ((2*I[1:-1]-I[:-2]-I[2:])**2).sum()/(len(I)*EPI**2)  # EPI = I.max()-baseline_apex
```

## Evaluation signals

- Every computed zigzag index falls within the paper's stated dynamic range 0–4.0
- Synthetic high-background and single-peak EICs (the cases where MCQ fails) yield low (good-quality) zigzag values, reproducing the paper's reported behavior
- Index is normalized by N * EPI^2, so it is scale-invariant to absolute peak intensity and independent of EIC length
- Script is deterministic end-to-end: a fixed random seed in the synthetic EIC generator reproduces identical index values across runs
- Filtering step removes 'bad' EICs before chromatographic peak detection, matching the four-module pipeline order

## Limitations

- Setting an optimal cutoff threshold for each quality metric still is a challenge task, so the index should be combined with other metrics rather than used alone
- The paper's LC/MS profile dataset and Matlab scripts are available only upon request; no public repository provided
- No license specified for the software (License: None for usage)
- Validation relies on ground truth peaks defined as peaks detected by at least two of XCMS, MZmine, and MAVEN, which is an indirect standard

## Evidence

- [abstract] A proposed EIC-related global zigzag index, based on an EIC's first order derivatives, is robust enough to evaluate EIC qualities with high background signals or with only a single peak.: "Our proposed EIC related global zigzag index is robust enough to evaluate EIC qualities in both scenarios."
- [abstract] MCQ index limitations that motivate the zigzag index: "Although the MCQ index is suited for selecting and aligning analyte components, it cannot fairly evaluate EICs with high background signals or those containing only a single peak."
- [abstract] Zigzag index among best-performing peak metrics due to chromatographic zigzag nature: "Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks."
- [methods] Pipeline placement of EIC quality evaluation and filtering: "which consists of four sequential modules: 1) acute EIC extraction, 2) EIC quality evaluation and filtering, 3) chromatographic peak detection, and 4) peak quality evaluation and filtering"
- [discussion] Cutoff threshold selection remains unsolved: "setting an optimal cutoff threshold for each metric still is a challenge task"
