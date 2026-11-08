---
name: retention-time-drift-correction
description: Use when integrating multiple untargeted LC/MS serum studies (e.g., HILIC
  positive-mode mzML files) whose RT and intensity drifts from batch effects and run
  order prevent cross-study feature alignment.
when_to_use_negative:
- Input is already a processed, RT-aligned feature table and the goal is downstream
  modeling (e.g., VAE fine-tuning) rather than signal detection and alignment
- Data are not MS1-level untargeted LC/MS metabolomics — mzLearn outputs are at the
  ion MS1 level and do not perform de-isotoping or MS/MS identification
- The analysis targets LC modes other than HILIC positive mode; the pre-trained model
  was developed specifically for HILIC positive mode and other modes are not covered
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mzLearn
  role: Data-driven MS1 signal detection and alignment; generates synthetic QC samples
    and learns RT/intensity drift maps iteratively from mzML files without user-set
    parameters
  repo: github.com/ReviveMed/mzEmbed
- name: metabCombiner
  role: Fits retention-time alignment splines across studies to correct inter-study
    RT drift
- name: Eclipse
  role: Cross-study subalignment; aggregates features across studies via graph-based
    grouping before RT spline correction
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:85ae3c45a9b73caaec3f33a328fa77d1dfb9ba2e29c344954e0c4bfbfeb2b136
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  tools:
  - Eclipse
  - metabCombiner
  - mzLearn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# retention-time-drift-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Corrects retention-time and intensity drift in untargeted LC/MS metabolomics by learning drift maps data-driven from mzML files (mzLearn) and fitting cross-study RT alignment splines (metabCombiner), without requiring experimental QC samples or user-set parameters.

## When to use

Use when integrating multiple untargeted LC/MS serum studies (e.g., HILIC positive-mode mzML files) whose RT and intensity drifts from batch effects and run order prevent cross-study feature alignment.

## When NOT to use

- Input is already a processed, RT-aligned feature table and the goal is downstream modeling (e.g., VAE fine-tuning) rather than signal detection and alignment
- Data are not MS1-level untargeted LC/MS metabolomics — mzLearn outputs are at the ion MS1 level and do not perform de-isotoping or MS/MS identification
- The analysis targets LC modes other than HILIC positive mode; the pre-trained model was developed specifically for HILIC positive mode and other modes are not covered

## Inputs

- Raw LC/MS mzML files from untargeted metabolomics runs (e.g., the 22 public HILIC positive-mode serum studies in Supplementary Table 5, downloadable from MetaboLights)
- Per-study mzLearn feature tables (2D table of features defined by median rt and m/z with normalized intensities)

## Outputs

- RT-corrected, cross-study aligned peak groups scored for robustness (2,736 robust peak groups when reproducing the paper's 22-dataset integration)
- A standardized feature matrix of floats with shape [n_samples, 2736] suitable for downstream VAE pretraining

## How to apply

Run mzLearn's data-driven MS1 signal detection on each study's mzML files: it requires no user-set parameters, generates synthetic QC samples, and iteratively learns internal thresholds plus RT and intensity drift maps from the data. Apply standard scaling to per-study intensity matrices so features are comparable across studies, then perform cross-study subalignment with Eclipse, which aggregates features via graph-based grouping. Fit retention-time alignment splines across studies with metabCombiner to correct inter-study RT drift. Finally, filter aligned peak groups by robustness score, dropping low-scoring peaks so only signals reproducible across studies survive; on the 22-study HILIC+ pretraining set this yielded 2,736 robust peak groups from 20,548 blood samples.

## Related tools

- **mzLearn** (Data-driven MS1 signal detection and alignment; generates synthetic QC samples and learns RT/intensity drift maps iteratively from mzML files without user-set parameters) — github.com/ReviveMed/mzEmbed
- **metabCombiner** (Fits retention-time alignment splines across studies to correct inter-study RT drift)
- **Eclipse** (Cross-study subalignment; aggregates features across studies via graph-based grouping before RT spline correction)

## Evaluation signals

- Output feature table retains the README-defined schema: features keyed by median rt and m/z with normalized intensities per sample
- After robustness-score filtering on the 22 HILIC+ studies, the number of retained peak groups matches the reported 2,736 robust metabolite signals (Supplementary Fig. 6)
- Downstream matrix has the expected [n_samples, 2736] float shape with z-scored values, loadable via pandas.read_csv(..., index_col=0)
- Drift correction proceeds without experimental QC samples — verify no user-set thresholds or external QC injections were required
- Reproducibility check: mzLearn is deterministic, so re-running on identical mzML inputs yields identical aligned feature tables

## Limitations

- Public metabolomics repositories lack standardized QC samples to estimate instrument drift, so drift maps are estimated from synthetic QC samples generated by the algorithm
- Iterative learning is computationally expensive — analysis of large cohorts can take up to 2.5 days on a single machine, and parallel processing on multiple compute nodes is not yet implemented
- mzLearn results are at the ion MS1 level only; de-isotoping, MS/MS identification, and mapping to MS/MS spectra must be carried out with downstream workflows

## Evidence

- [readme] README: mzLearn autonomously corrects RT and intensity drift: [evidence span withheld — anchor status: external_doc]
- [other] Source card workflow: metabCombiner RT splines and robustness filtering: [evidence span withheld — anchor status: not_found]
