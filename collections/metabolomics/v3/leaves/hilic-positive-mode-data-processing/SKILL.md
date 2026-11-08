---
name: hilic-positive-mode-data-processing
description: 'Use when you have raw mzML files from HILIC positive-mode serum/plasma
  LC/MS studies (ideally multiple studies) and need a feature matrix suitable for
  downstream ML: signal detection without manual parameters, cross-study alignment,
  RT drift correction, and filtering to reproducible features.'
when_to_use_negative:
- The data are negative-mode, reversed-phase, or otherwise non-HILIC-positive LC/MS
  — the article states the current pre-trained model was developed specifically for
  HILIC positive mode.
- You need de-isotoping or MS/MS-level compound identification — mzLearn outputs are
  limited to the ion MS1 level, and mapping to MS/MS spectra must be carried out with
  downstream workflows.
- You already have a processed feature table and only need downstream annotation —
  this skill covers raw-mzML-to-feature-matrix generation and alignment, not annotation
  itself.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0153
- http://edamontology.org/topic_3474
tools:
- name: mzLearn
  role: Zero-parameter data-driven MS1 signal detection, RT/intensity drift correction,
    and feature-table generation from raw mzML files
- name: Eclipse
  role: Cross-study subalignment/aggregation of features across studies via graph-based
    grouping
- name: metabCombiner
  role: Fits retention-time alignment splines across studies to correct inter-study
    RT drift
- name: mzEmbed
  role: Framework for pretraining and fine-tuning the VAE on the standardized 2,736-feature
    matrix
  repo: github.com/ReviveMed/mzEmbed
- name: XCMS
  role: Baseline feature-detection comparator for validating mzLearn's signal counts
    and TP/FP rates
- name: ASARI
  role: Baseline feature-detection comparator for validating mzLearn's signal counts
    and TP/FP rates
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:56764efcc9a8e07a468bbf0e1baf76b87df4a500a501352919c87b3c845e839a
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
  - Eclipse
  - XCMS
  - metabCombiner
  - mzEmbed
  - mzLearn
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0153
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# hilic-positive-mode-data-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A cross-study processing and pretraining pipeline for HILIC positive-mode LC/MS serum untargeted metabolomics: raw mzML files are run through mzLearn's zero-parameter signal detection, aligned across studies, filtered to robust peak groups, and used to pretrain a VAE whose encoder can be fine-tuned on new cohorts.

## When to use

Use when you have raw mzML files from HILIC positive-mode serum/plasma LC/MS studies (ideally multiple studies) and need a feature matrix suitable for downstream ML: signal detection without manual parameters, cross-study alignment, RT drift correction, and filtering to reproducible features.

## When NOT to use

- The data are negative-mode, reversed-phase, or otherwise non-HILIC-positive LC/MS — the article states the current pre-trained model was developed specifically for HILIC positive mode.
- You need de-isotoping or MS/MS-level compound identification — mzLearn outputs are limited to the ion MS1 level, and mapping to MS/MS spectra must be carried out with downstream workflows.
- You already have a processed feature table and only need downstream annotation — this skill covers raw-mzML-to-feature-matrix generation and alignment, not annotation itself.

## Inputs

- Raw LC/MS mzML files from HILIC positive-mode serum/plasma untargeted metabolomics runs
- CSV feature matrices of floats shaped [n_samples, 2736] (X_Pretrain_Discovery_Train.csv, X_Pretrain_Discovery_Val.csv, X_Pretrain_Test.csv; pandas index_col=0)
- Optional label CSVs (e.g. IMDC class, overall survival time/event) for later task fine-tuning

## Outputs

- Two-dimensional feature table of detected MS1 ion features defined by median rt and m/z values with normalized intensities across samples
- Robust cross-study peak-group matrix — 2,736 robust metabolite signals in the reference build
- Pretrained VAE artifacts (best_model.pt, config.json, study.html, latent plots) plus exported encoder.pt and Z_Pretrain_*.csv latent embeddings

## How to apply

Download mzML files for the source studies (e.g. the 22 public HILIC positive-mode pretraining studies from MetaboLights, Supplementary Table 5) and run mzLearn's data-driven MS1 signal detection, which requires no user-set parameters and iteratively learns internal thresholds and drift maps via synthetic QC samples. Apply standard scaling to per-study intensity matrices to make features comparable across studies, then aggregate features across studies with cross-study subalignment (Eclipse) and fit retention-time alignment splines (metabCombiner) to correct inter-study RT drift. Filter aligned peak groups by robustness score to retain only signals reproducible across studies — the reference run yielded 2,736 robust peak groups from 20,548 blood samples. Use the resulting standardized [n_samples, 2736] CSV matrices (z-scored, with X_Pretrain_Discovery_Train/Val/Test splits) as input to the mzEmbed VAE pretraining; optionally export the encoder with get_pretrain_encoder for later fine-tuning.

## Related tools

- **mzLearn** (Zero-parameter data-driven MS1 signal detection, RT/intensity drift correction, and feature-table generation from raw mzML files)
- **Eclipse** (Cross-study subalignment/aggregation of features across studies via graph-based grouping)
- **metabCombiner** (Fits retention-time alignment splines across studies to correct inter-study RT drift)
- **mzEmbed** (Framework for pretraining and fine-tuning the VAE on the standardized 2,736-feature matrix) — github.com/ReviveMed/mzEmbed
- **XCMS** (Baseline feature-detection comparator for validating mzLearn's signal counts and TP/FP rates)
- **ASARI** (Baseline feature-detection comparator for validating mzLearn's signal counts and TP/FP rates)

## Examples

```
python -m mz_embed.pretrain.run_pretrain_VAE_main --data_location /path/to/pretrain_csvs --latent_size 460 --n_layers 3 --trials 40 --max_epochs 200 --output_dir /path/to/out_pretrain
```

## Evaluation signals

- Feature-table schema: two-dimensional table of detected features defined by median rt and m/z values, with normalized intensities across samples
- Robust peak-group count in the expected range: the reference cross-study integration yielded 2,736 robust metabolite signals across 22 studies / 20,548 blood samples
- Signal-detection sensitivity check: mzLearn averages ~11,442 signals per dataset vs 7,100 (XCMS) and 4,655 (ASARI), with higher TP (89.0%) and lower FP (12.5%) — counts should not collapse toward baseline-tool levels
- Pretraining inputs must be floats, z-scored, shape [n_samples, 2736], loaded with pandas.read_csv(..., index_col=0)
- Fine-tuning with --pretrained_dir should outperform random initialization on the same cohort (e.g. IMDC binary AUC ~0.93, OS C-index ~0.67 in the reference evaluation)

## Limitations

- Iterative learning is computationally expensive — analysis can take up to 2.5 days on a single machine; multi-node parallel processing is not yet implemented.
- The pretrained model is specific to HILIC positive mode; other LC types/modes require multi-modal learning that is not yet developed.
- Public metabolomics repositories often lack standardized QC samples, complicating instrument-drift estimation; batch covariates or domain-invariant regularization are not yet incorporated.
- The ICI-response classification and fine-tuned models need additional validation for clinical application and generalizability across multiple fine-tuning datasets is untested.

## Evidence

- [readme] Zero-parameter design: [evidence span withheld — anchor status: external_doc]
- [readme] README invocation of pretraining: [evidence span withheld — anchor status: external_doc]
