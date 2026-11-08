---
name: lcms-ms1-signal-detection
description: Use when you have raw LC/MS untargeted metabolomics data in mzML format
  (e.g., HILIC positive-mode serum studies) and need to detect metabolite signals
  without user-set parameters, QC samples, or manual threshold tuning.
when_to_use_negative:
- When you need de-isotoping, MS/MS identification, or compound annotation — mzLearn
  outputs are limited to the ion MS1 level and does not perform these steps.
- When working with LC modes other than HILIC positive mode (e.g., reversed-phase
  or negative mode), since the current pre-trained model and mzLearn pretraining set
  were developed specifically for HILIC positive mode.
- When a parameterized classical pipeline is required for reproducibility audits that
  demand explicit user-set thresholds, as mzLearn is designed to be zero-parameter.
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: mzLearn
  role: Primary data-driven MS1 signal-detection and alignment algorithm that runs
    from mzML files without user-set parameters; iteratively learns signal characteristics
    and drift maps to produce feature tables.
  repo: github.com/ReviveMed/mzEmbed
- name: XCMS
  role: Baseline peak-detection tool used as a comparison benchmark; mzLearn detects
    11,442 signals on average vs 7,100 for XCMS.
- name: ASARI
  role: Baseline peak-detection tool used as a comparison benchmark; mzLearn detects
    11,442 signals on average vs 4,655 for ASARI.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:db32cfca305ce111b31cdbed7aa439721f769691fb9f929995aba1fd59bc22b2
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
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# lcms-ms1-signal-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

mzLearn is a data-driven, zero-parameter MS1 signal-detection and alignment algorithm for LC/MS untargeted metabolomics that runs directly on mzML files, iteratively learns signal characteristics and drift maps via synthetic QC samples, and outperforms XCMS and ASARI in sensitivity and false-positive control.

## When to use

Use when you have raw LC/MS untargeted metabolomics data in mzML format (e.g., HILIC positive-mode serum studies) and need to detect metabolite signals without user-set parameters, QC samples, or manual threshold tuning.

## When NOT to use

- When you need de-isotoping, MS/MS identification, or compound annotation — mzLearn outputs are limited to the ion MS1 level and does not perform these steps.
- When working with LC modes other than HILIC positive mode (e.g., reversed-phase or negative mode), since the current pre-trained model and mzLearn pretraining set were developed specifically for HILIC positive mode.
- When a parameterized classical pipeline is required for reproducibility audits that demand explicit user-set thresholds, as mzLearn is designed to be zero-parameter.

## Inputs

- Raw LC/MS untargeted metabolomics data files in the open-source mzML format
- Large-scale dataset batches (e.g., up to 2,075 files in a single run)

## Outputs

- A two-dimensional table of detected ion features defined by median rt and m/z values, with normalized intensities across samples
- MS1 ion-feature matrices suitable for downstream de-isotoping, annotation, MS/MS confirmation, and pre-training generative models (e.g., VAE)

## How to apply

Convert raw LC/MS files to the open-source mzML format, then run mzLearn's data-driven MS1 signal detection, which requires no user-set parameters and iteratively learns internal thresholds and drift maps from the data. mzLearn generates synthetic QC samples and constructs intensity and retention-time drift maps through a dynamic, iterative, data-driven learning process, correcting for rt and intensity drifts caused by batch effects and run order without experimental QC samples. The output is a two-dimensional table of detected ion features defined by median rt and m/z values with normalized intensities across samples. Compared to XCMS and ASARI, mzLearn detects ~60% more signals (11,442 average vs 7,100/4,655) with higher TP rate (89.0% vs 77.4%/49.6%) and lower FP rate (12.5% vs 17.3%/18.8%). Note that mzLearn outputs are limited to the ion MS1 level and do not perform de-isotoping or MS/MS identification, which should be handled by downstream workflows.

## Related tools

- **mzLearn** (Primary data-driven MS1 signal-detection and alignment algorithm that runs from mzML files without user-set parameters; iteratively learns signal characteristics and drift maps to produce feature tables.) — github.com/ReviveMed/mzEmbed
- **XCMS** (Baseline peak-detection tool used as a comparison benchmark; mzLearn detects 11,442 signals on average vs 7,100 for XCMS.)
- **ASARI** (Baseline peak-detection tool used as a comparison benchmark; mzLearn detects 11,442 signals on average vs 4,655 for ASARI.)

## Examples

```
python -m mz_embed.pretrain.run_pretrain_VAE_main --data_location /path/to/pretrain_csvs --latent_size 460 --n_layers 3 --trials 40 --max_epochs 200 --output_dir /path/to/out_pretrain
```

## Evaluation signals

- Across public datasets, the mean number of detected signals should be on the order of 11,442 — substantially higher than XCMS (~7,100) and ASARI (~4,655) baselines.
- On labeled validation data, true-positive rate should approach ~89.0% and false-positive rate should remain near ~12.5%, outperforming XCMS (77.4%/17.3%) and ASARI (49.6%/18.8%).
- Retention-time and intensity drifts across batch/run-order should be visibly corrected in the resulting feature table, with no experimental QC samples required.
- The output must be a two-dimensional feature table keyed by median rt and m/z with normalized intensities across samples, suitable for downstream VAE pre-training.
- For the 22 public HILIC positive-mode studies (20,548 blood samples), the downstream robust-signal filtering should yield 2,736 robust metabolite signals.

## Limitations

- mzLearn outputs are limited to the ion MS1 level; it does not perform de-isotoping or MS/MS identification, and no MS/MS integration into pretraining is implemented.
- The current pre-trained model is specific to HILIC positive mode; other LC modes and modalities are not covered.
- Iterative learning increases computational time (up to 2.5 days on a single machine); parallel processing on multiple compute nodes is not yet implemented.
- Public metabolomics repositories lack standardized QC samples to estimate instrument drift.
- Multi-modal learning for integrating various LC types and modes is not yet developed.

## Evidence

- [readme] output table definition from README: [evidence span withheld — anchor status: external_doc]
- [readme] zero-parameter design from README: [evidence span withheld — anchor status: not_found]
- [readme] iterative drift correction from README: [evidence span withheld — anchor status: external_doc]
- [readme] repository URL: [evidence span withheld — anchor status: external_doc]
