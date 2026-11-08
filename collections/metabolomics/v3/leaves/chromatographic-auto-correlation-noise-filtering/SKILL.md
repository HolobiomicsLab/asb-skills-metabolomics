---
name: chromatographic-auto-correlation-noise-filtering
description: Use when raw LC/MS data in mzML format must be converted into an MS1
  ion-feature table without manually tuned detection thresholds, and noise must be
  separated from true chromatographic peaks in a data-driven, parameter-free way.
when_to_use_negative:
- The input is already a feature/peak table rather than raw mzML LC/MS data — this
  skill operates on raw MS1 spectra from mzML files
- De-isotoping, metabolite annotation, or MS/MS identification is required — mzLearn
  results are at the ion MS1 level and it does not perform de-isotoping or MS/MS identification
- User-set parameters or expert-tuned thresholds are desired or mandated by an existing
  validated pipeline (e.g., a tuned XCMS or ASARI workflow that must be reproduced
  as-is)
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: mzLearn
  role: 'Implements the patch-based auto/cross-correlation signal-detection and alignment
    core: parses mzML files, filters patches by auto-correlation, aligns by sliding
    cross-correlation, and iteratively learns thresholds from ''pin'' patches'
  repo: https://github.com/ReviveMed/mzEmbed
- name: XCMS
  role: Baseline signal-detection method used for benchmark comparison (7,100 signals
    on average vs mzLearn's 11,442)
- name: ASARI
  role: Baseline signal-detection method used for benchmark comparison (4,655 signals
    on average vs mzLearn's 11,442)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:3b8fa81de661cd56fc0eddde053754932b7a27f18dca6347552ab1ede72dd0d3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ReviveMed/mzEmbed
  repo_url: https://github.com/ReviveMed/mzEmbed
  tools:
  - ASARI
  - XCMS
  - mzLearn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# chromatographic-auto-correlation-noise-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

mzLearn's data-driven MS1 signal-detection core: it partitions rt/m/z space into dynamically resized patches, separates true chromatographic peaks from noise via auto-correlation, and learns its own thresholds from highest-quality 'pin' patches over ~3 iterations, requiring no user-set parameters or QC samples.

## When to use

Use when raw LC/MS data in mzML format must be converted into an MS1 ion-feature table without manually tuned detection thresholds, and noise must be separated from true chromatographic peaks in a data-driven, parameter-free way.

## When NOT to use

- The input is already a feature/peak table rather than raw mzML LC/MS data — this skill operates on raw MS1 spectra from mzML files
- De-isotoping, metabolite annotation, or MS/MS identification is required — mzLearn results are at the ion MS1 level and it does not perform de-isotoping or MS/MS identification
- User-set parameters or expert-tuned thresholds are desired or mandated by an existing validated pipeline (e.g., a tuned XCMS or ASARI workflow that must be reproduced as-is)

## Inputs

- Raw LC/MS data files in mzML format (MS1 spectral data)
- Large cohorts of LC/MS runs without experimental QC samples (e.g., thousands of samples; the paper demonstrates runs of 2,075 files)

## Outputs

- Two-dimensional table of detected MS1 ion features defined by median rt and m/z values, with normalized intensities across samples
- Retention-time drift map built by sliding cross-correlation alignment across files
- Internally learned intensity and rt drift maps / synthetic QC samples produced during iterative learning

## How to apply

Load the MS1 spectral data from the input mzML files using mzLearn's mzML parsing utilities, then recursively partition the retention-time and m/z space into patches that are dynamically resized according to local signal density. Apply auto-correlation filtering to each patch to distinguish true chromatographic peaks from noise, and estimate the correlation thresholds via the iterative 'pin'-based procedure in which thresholds are learned from the highest-quality pin patches until convergence (~3 iterations). Perform sliding cross-correlation alignment across files to build the retention-time drift map and match signals between samples, simultaneously correcting rt and intensity drifts caused by batch effects and run order. Assemble the filtered, aligned signals into a final two-dimensional feature table defined by median rt and m/z values with normalized intensities across samples. The rationale is that mzLearn is deterministic and non-parametric: it iteratively refines signal characteristics and filters out noise to recover missing true peaks, generating synthetic QC samples and drift maps through a dynamic, iterative, data-driven learning process rather than relying on experimental QC injections.

## Related tools

- **mzLearn** (Implements the patch-based auto/cross-correlation signal-detection and alignment core: parses mzML files, filters patches by auto-correlation, aligns by sliding cross-correlation, and iteratively learns thresholds from 'pin' patches) — https://github.com/ReviveMed/mzEmbed
- **XCMS** (Baseline signal-detection method used for benchmark comparison (7,100 signals on average vs mzLearn's 11,442))
- **ASARI** (Baseline signal-detection method used for benchmark comparison (4,655 signals on average vs mzLearn's 11,442))

## Evaluation signals

- Convergence of the iterative pin-threshold estimation after approximately 3 iterations
- Per-dataset signal counts consistent with the algorithm's reported behavior: 11,442 signals on average across 15 public datasets (vs 7,100 for XCMS and 4,655 for ASARI)
- Output schema check: a two-dimensional table of features defined by median rt and m/z values with normalized intensities across samples
- Benchmark TP/FP quality: expected ~89.0% true positive rate and ~12.5% false positive rate, higher TP and lower FP than XCMS (77.4%/17.3%) and ASARI (49.6%/18.8%)
- No user-set parameters or QC samples were supplied at run time; thresholds and drift maps must be learned entirely from the data

## Limitations

- Iterative learning is computationally expensive — analysis can take up to 2.5 days on a single machine, and parallel processing on multiple compute nodes is not yet implemented
- mzLearn outputs are limited to the ion MS1 level; it does not perform de-isotoping or MS/MS identification, and no MS/MS integration is implemented
- Public metabolomics repositories often lack standardized QC samples to estimate instrument drift (mzLearn compensates via synthetic QC samples, but external drift validation may be limited)

## Evidence

- [abstract] Parameter-free MS1 detection from mzML: "we introduce mzLearn, a data-driven MS1 signal-detection and alignment method that runs from mzML files without user-set parameters"
- [discussion] Iterative noise filtering rationale: "it iteratively refines signal characteristics and filters out the noise to identify the missing true peaks"
- [readme] Drift correction during iterative detection: [evidence span withheld — anchor status: external_doc]
- [readme] Zero-parameter, no-QC design: [evidence span withheld — anchor status: not_found]
- [abstract] Signal-count benchmark vs baselines: "Across 15 public datasets, mzLearn detects 11,442 signals on average vs 7,100 (XCMS) and 4,655 (ASARI)"
- [abstract] Detection accuracy benchmark: "with higher TP (89.0% vs 77.4% vs 49.6%) and lower FP (12.5% vs 17.3% vs 18.8%)"
- [readme] Output schema of detected features: [evidence span withheld — anchor status: not_found]
