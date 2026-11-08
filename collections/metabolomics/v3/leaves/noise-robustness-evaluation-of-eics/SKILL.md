---
name: noise-robustness-evaluation-of-eics
description: Use when LC/MS metabolomics extracted ion chromatograms (EICs) must be
  scored or filtered for quality — especially EICs with high background signals or
  only a single peak, where the MCQ index cannot fairly evaluate them.
when_to_use_negative:
- Input is a set of already-detected chromatographic peaks rather than full EIC traces
  — the paper's six peak quality metrics (sharpness, Gaussian similarity, SNR, peak
  significance level, TPASR, local peak-related zigzag index) apply to peaks instead.
- The goal is selecting and aligning analyte components for which the MCQ index is
  adequate — this skill targets the high-background and single-peak EICs that the
  MCQ index cannot fairly evaluate.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: Matlab
  role: Language of the authors' own data processing program, whose module 2 performs
    EIC quality evaluation and filtering using the global zigzag index
- name: Python
  role: Language for a self-contained deterministic reimplementation of the zigzag
    index function plus the synthetic EIC generator (normal, high-background, single-peak
    cases)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/1471-2105-15-s11-s5
    title: Quality evaluation of extracted ion chromatograms and chromatographic peaks
      in liquid chromatography/mass spectrometry-based metabolomics data
  source_content_hash: sha256:b45be549cdcf6b9afc7a00616a01b17862b38f2a0ef4ea1be2221c4b7d7f60a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Matlab
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# noise-robustness-evaluation-of-eics

## Summary

Implements the paper's EIC-related global zigzag index — a normalized first-order-derivative score with dynamic range 0–4.0 — to evaluate the quality of extracted ion chromatograms (EICs) in LC/MS-based metabolomics data, remaining robust for EICs with high background signals or only a single peak, where the mass chromatographic quality (MCQ) index fails.

## When to use

Use when LC/MS metabolomics extracted ion chromatograms (EICs) must be scored or filtered for quality — especially EICs with high background signals or only a single peak, where the MCQ index cannot fairly evaluate them.

## When NOT to use

- Input is a set of already-detected chromatographic peaks rather than full EIC traces — the paper's six peak quality metrics (sharpness, Gaussian similarity, SNR, peak significance level, TPASR, local peak-related zigzag index) apply to peaks instead.
- The goal is selecting and aligning analyte components for which the MCQ index is adequate — this skill targets the high-background and single-peak EICs that the MCQ index cannot fairly evaluate.

## Inputs

- Extracted ion chromatogram (EIC) intensity arrays (ordered intensity vectors I_1..I_N) extracted from LC/MS profile data (e.g., the Agilent HPLC / Q-TOF Premier profile dataset used in the paper)
- Synthetic EIC arrays generated with a fixed random seed and documented peak width, height, background and noise parameters (normal, high-background, and single-peak cases) for validation

## Outputs

- A scalar EIC-related global zigzag index per EIC (dimensionless, dynamic range 0–4.0)
- Per-case printed index values plus assertions confirming the 0–4.0 range and low (good-quality) values for the high-background and single-peak synthetic EICs
- A self-contained deterministic script (Matlab or Python) implementing the zigzag function and the three synthetic EIC test cases

## How to apply

For each extracted EIC (an ordered intensity vector I_1…I_N from the LC/MS profile data), compute the effective peak intensity EPI = apex maximum intensity minus the baseline at the apex, then compute the summed local zigzag term over first-order increments, normalized as zigzag_index = Σ(2I_n − I_{n−1} − I_{n+1})² / (N × EPI²), whose dynamic range is 0–4.0. Validate the implementation with a deterministic synthetic EIC generator (fixed random seed; documented peak width, height, background and noise parameters) producing three cases: a normal EIC, an EIC with high background signal, and a single-peak EIC. Assert that every index value falls within 0–4.0 and that the high-background and single-peak cases yield low (good-quality) values — precisely the scenarios where the MCQ index cannot fairly evaluate EICs. In the authors' Matlab program this metric occupies module 2 of four sequential modules (acute EIC extraction → EIC quality evaluation and filtering → chromatographic peak detection → peak quality evaluation and filtering), i.e., it filters 'bad' EICs before the downstream time-consuming peak detection processing. Reimplement the function as a self-contained deterministic script in Matlab or Python.

## Related tools

- **Matlab** (Language of the authors' own data processing program, whose module 2 performs EIC quality evaluation and filtering using the global zigzag index)
- **Python** (Language for a self-contained deterministic reimplementation of the zigzag index function plus the synthetic EIC generator (normal, high-background, single-peak cases))

## Examples

```
import numpy as np; zzi = ((2*I[1:-1]-I[:-2]-I[2:])**2).sum()/(len(I)*EPI**2)  # EPI = apex max intensity minus baseline at apex
```

## Evaluation signals

- All computed zigzag index values fall within the paper's stated dynamic range of 0–4.0.
- The high-background EIC and the single-peak EIC both yield low (good-quality) index values — the two scenarios in which the MCQ index fails.
- Rerunning the script with the same fixed random seed and documented peak width, height, background and noise parameters reproduces identical index values (determinism).
- The denominator uses N × EPI² with EPI = max intensity minus baseline at the apex, so index values are invariant to uniform intensity scaling of an EIC.
- The metric is placed as module 2 of the four-module pipeline — after acute EIC extraction and before chromatographic peak detection — matching the article's workflow.

## Limitations

- The authors' LC/MS profile dataset and Matlab scripts are available only upon request; no public repository is provided and no license is specified ('License: None for usage'), so the metric must be reimplemented from the paper.
- Setting an optimal cutoff threshold for the zigzag index (and for each quality metric) remains an unsolved challenge according to the article.
- Machine learning-based automatic peak filtering (e.g., SVM) was not implemented; it would require a curated training set of 'good' and 'bad' representative peaks that does not yet exist.

## Evidence

- [abstract] The EIC-related global zigzag index is robust enough to evaluate EIC qualities in both problematic scenarios: "Our proposed EIC related global zigzag index is robust enough to evaluate EIC qualities in both scenarios."
- [abstract] The MCQ index cannot fairly evaluate EICs with high background signals or only a single peak, motivating the zigzag index: "Although the MCQ index is suited for selecting and aligning analyte components, it cannot fairly evaluate EICs with high background signals or those containing only a single peak."
- [abstract] Among the six peak quality metrics, sharpness, peak significance level and zigzag index perform best: "Of the six peak quality evaluation metrics, the sharpness, peak significance level, and zigzag index outperform the others due to the zigzag nature of LC/MS chromatographic peaks."
- [methods] The authors developed their own Matlab data processing program containing the EIC quality evaluation module: "we developed our own data processing program in Matlab, which consists of four sequential modules"
- [methods] Pipeline order: acute EIC extraction, EIC quality evaluation and filtering, chromatographic peak detection, peak quality evaluation and filtering: "2) EIC quality evaluation and filtering, 3) chromatographic peak detection, and 4) peak quality evaluation and filtering"
- [discussion] Optimal per-metric cutoff thresholds remain an open challenge: "setting an optimal cutoff threshold for each metric still is a challenge task"
- [discussion] The dataset and Matlab scripts are only available upon request, so implementations must be reconstructed: "The LC/MS profile dataset and Matlab scripts are available upon request."
