---
name: cross-correlation-rt-drift-alignment
description: Use when raw LC/MS data in mzML format must be converted into a drift-corrected
  MS1 feature table without user-tuned parameters, QC samples, or prior knowledge
  of retention-time drift.
when_to_use_negative:
- The input is already a feature table (e.g., the CSV matrices consumed by mzEmbed
  pretraining/finetuning) rather than raw mzML files — mzLearn's detection/alignment
  stage operates on raw LC/MS data
- The goal includes de-isotoping, compound annotation, or MS/MS identification — mzLearn
  outputs are limited to the ion MS1 level and these must be handled by downstream
  workflows
- 'You need a fast, interactive run: iterative learning can take up to 2.5 days on
  a single machine with no parallel multi-node processing implemented'
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: mzLearn
  role: 'Primary engine: performs the patch partitioning, auto-correlation filtering,
    sliding cross-correlation alignment, and pin-based threshold learning directly
    from mzML files without user-set parameters'
  repo: github.com/ReviveMed/mzEmbed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:1f7ff439f3442cdae97dc33435d7bf633dd03fab0a6d589516fd0476f26f1c24
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# cross-correlation-rt-drift-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

mzLearn's data-driven MS1 signal detection and alignment core: it recursively partitions rt/m/z space into dynamically resized patches, filters them by auto-correlation, aligns them across LC/MS files by sliding cross-correlation, and learns thresholds from highest-quality 'pin' patches — requiring no user-set parameters or QC samples.

## When to use

Use when raw LC/MS data in mzML format must be converted into a drift-corrected MS1 feature table without user-tuned parameters, QC samples, or prior knowledge of retention-time drift.

## When NOT to use

- The input is already a feature table (e.g., the CSV matrices consumed by mzEmbed pretraining/finetuning) rather than raw mzML files — mzLearn's detection/alignment stage operates on raw LC/MS data
- The goal includes de-isotoping, compound annotation, or MS/MS identification — mzLearn outputs are limited to the ion MS1 level and these must be handled by downstream workflows
- You need a fast, interactive run: iterative learning can take up to 2.5 days on a single machine with no parallel multi-node processing implemented

## Inputs

- Raw LC/MS data files in open-source mzML format (demonstrated at scale, e.g., 2,075 files in a single run)
- No user-set parameters, prior knowledge, or experimental QC samples are required as input

## Outputs

- Two-dimensional MS1 ion-feature table defined by median rt and m/z values with normalized intensities across samples
- Retention-time drift map used to align signals across files and cohorts
- Internal learned thresholds (auto-correlation/cross-correlation cutoffs derived from 'pin' patches)

## How to apply

Load MS1 spectral data from each input mzML file using mzLearn's mzML parsing utilities, then recursively partition the retention-time and m/z space into patches that are dynamically resized according to local signal density. Apply auto-correlation filtering to each patch to separate true chromatographic peaks from noise, and estimate correlation thresholds via the iterative 'pin'-based procedure in which thresholds are learned from the highest-quality pin patches until convergence (approximately three iterations). Next, perform sliding cross-correlation alignment across files to build the retention-time drift map and match signals between samples, which simultaneously corrects rt and intensity drifts caused by batch effects and run order. The rationale is that the algorithm is deterministic and non-parametric — it iteratively learns internal thresholds and drift maps from the data itself, so no manual cutoffs (e.g., ppm tolerances or rt windows) need to be set. Finally, assemble the filtered and aligned signals into a two-dimensional feature table defined by median rt and m/z values with normalized intensities across samples.

## Related tools

- **mzLearn** (Primary engine: performs the patch partitioning, auto-correlation filtering, sliding cross-correlation alignment, and pin-based threshold learning directly from mzML files without user-set parameters) — github.com/ReviveMed/mzEmbed

## Evaluation signals

- Iterative parameter learning converges after approximately three iterations, outputting a final high-quality feature list with minimal missing values aligned across samples
- Across 15 public datasets, per-dataset signal counts are consistent with the algorithm's reported behavior (~11,442 signals on average vs 7,100 for XCMS and 4,655 for ASARI)
- True positive rate around 89.0% and false positive rate around 12.5% on benchmarked data, exceeding XCMS (77.4% TP, 17.3% FP) and ASARI (49.6% TP, 18.8% FP)
- Retention-time and intensity drifts caused by batch effects and run order are visibly corrected without experimental QC samples
- Determinism check: rerunning on the same mzML inputs reproduces identical thresholds and drift maps because the algorithm is deterministic and non-parametric

## Limitations

- Iterative learning increases computational time, with analysis taking up to 2.5 days on a single machine; parallel processing on multiple compute nodes is not yet implemented
- Public metabolomics repositories often lack standardized QC samples to estimate instrument drift, which the algorithm must infer purely from the data
- Results are at the ion MS1 level only — no de-isotoping, MS/MS identification, or MS/MS integration into downstream model pretraining
- The downstream pre-trained model built on mzLearn features was developed specifically for HILIC positive mode; other LC modes and modalities are not covered

## Evidence

- [abstract] a data-driven MS1 signal-detection and alignment method that runs from mzML files without user-set parameters: "we introduce mzLearn, a data-driven MS1 signal-detection and alignment method that runs from mzML files without user-set parameters"
- [discussion] mzLearn is a deterministic, non-parametric algorithm that iteratively learns internal thresholds and drift maps from the data: "mzLearn is a deterministic, non-parametric algorithm that iteratively learns internal thresholds and drift maps from the data"
- [discussion] constructing intensity and rt drift maps through a dynamic, iterative, data-driven learning process: "generating synthetic QC samples and constructing intensity and rt drift maps through a dynamic, iterative, data-driven learning process"
- [discussion] iteratively refines signal characteristics and filters out the noise to identify the missing true peaks: "it iteratively refines signal characteristics and filters out the noise to identify the missing true peaks"
- [readme] correcting for retention time (rt) and intensity drifts caused by batch effects and run order: [evidence span withheld — anchor status: external_doc]
- [readme] A two-dimensional table of detected features defined by median rt and m/z values, with normalized intensities across samples: [evidence span withheld — anchor status: external_doc]
