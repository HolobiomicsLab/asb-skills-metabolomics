---
name: ms1-ion-feature-matrix-preprocessing
description: Use when raw mzLearn output (a feature table of median rt/m/z-defined
  signals with normalized intensities) must be standardized and split into X_Pretrain_*/X_Finetune_*
  CSVs before VAE training.
when_to_use_negative:
- Input is already a z-scored feature matrix in the required [n, 2736] shape — re-standardizing
  or reshaping is redundant and risks altering pretrained-model-compatible values
- Starting from raw LC/MS mzML files rather than mzLearn output — signal detection
  must happen first (mzLearn is the zero-parameter MS1 detector; this skill only formats
  its table)
- The cohort uses an LC mode other than HILIC positive — the 2,736-feature set and
  pretrained weights were developed specifically for HILIC positive mode
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: mzLearn
  role: Produces the MS1 ion-feature matrix (median rt/m/z-defined signals, normalized
    intensities) that this preprocessing step formats and z-scores
  repo: https://github.com/ReviveMed/mzEmbed
- name: mzEmbed
  role: Consumes the preprocessed X_* CSVs; defines the expected [n_samples, 2736]
    float shape, z-scored values, and index_col=0 semantics
  repo: https://github.com/ReviveMed/mzEmbed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.01.26.634927
    title: mzLearn, a data-driven LC/MS signal detection algorithm, enables pre-trained
      generative models for enhanced patient stratification
  source_content_hash: sha256:372eb1ac2a23e24945cc0bbeee14045316dc5e65c7cfe97e4154ffe66e5a3618
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
  - VAE
  - mzEmbed
  - mzLearn
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# ms1-ion-feature-matrix-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess mzLearn-derived MS1 ion-feature matrices (e.g. 2,736 robust metabolite signals across 20,548 blood samples) — including z-score standardization of signal intensities and reformatting into train/val/test CSVs — so the matrix is ready for VAE pretraining or fine-tuning in mzEmbed.

## When to use

Use when raw mzLearn output (a feature table of median rt/m/z-defined signals with normalized intensities) must be standardized and split into X_Pretrain_*/X_Finetune_* CSVs before VAE training.

## When NOT to use

- Input is already a z-scored feature matrix in the required [n, 2736] shape — re-standardizing or reshaping is redundant and risks altering pretrained-model-compatible values
- Starting from raw LC/MS mzML files rather than mzLearn output — signal detection must happen first (mzLearn is the zero-parameter MS1 detector; this skill only formats its table)
- The cohort uses an LC mode other than HILIC positive — the 2,736-feature set and pretrained weights were developed specifically for HILIC positive mode

## Inputs

- mzLearn MS1 output table: detected ion features defined by median rt and m/z values, with normalized intensities across samples
- Sample demographic/clinical labels (disease, age, IMDC class, OS time/event)
- Study/split assignment for each sample (discovery-train, discovery-val, test)

## Outputs

- X_Pretrain_Discovery_Train.csv, X_Pretrain_Discovery_Val.csv, X_Pretrain_Test.csv — z-scored float matrices of shape [n_samples, 2736] with row index in first column
- X_Finetune_Train.csv, X_Finetune_Val.csv, X_Finetune_Test.csv for downstream cohorts
- Optional y_* label CSVs aligned to the row index

## How to apply

Take the mzLearn output — a two-dimensional table of detected MS1 ion features defined by median rt and m/z values with normalized intensities across samples — and convert it to a CSV of floats with shape [n_samples, 2736] (the robust peak set from 22 public studies / 20,548 blood samples). Standardize each feature column via z-scoring so the matrix is 'float; z-scored; [n, 2736]' as mzEmbed expects. Write the standardized matrix as separate split files using pandas.read_csv(..., index_col=0) semantics: X_Pretrain_Discovery_Train.csv, X_Pretrain_Discovery_Val.csv, X_Pretrain_Test.csv for pretraining, or X_Finetune_{Train,Val,Test}.csv for downstream cohorts. Attach task-specific label columns (disease, age, IMDC class, OS time/event) as optional y_* CSVs aligned by row index. Keep the 2,736-column feature axis consistent across all splits so pretrained encoder weights and latent embeddings map onto the same features.

## Related tools

- **mzLearn** (Produces the MS1 ion-feature matrix (median rt/m/z-defined signals, normalized intensities) that this preprocessing step formats and z-scores) — https://github.com/ReviveMed/mzEmbed
- **mzEmbed** (Consumes the preprocessed X_* CSVs; defines the expected [n_samples, 2736] float shape, z-scored values, and index_col=0 semantics) — https://github.com/ReviveMed/mzEmbed

## Examples

```
import pandas as pd; X = (pd.read_csv('mzlearn_features.csv', index_col=0) - pd.read_csv('mzlearn_features.csv', index_col=0).mean()) / pd.read_csv('mzlearn_features.csv', index_col=0).std(); X.to_csv('X_Pretrain_Discovery_Train.csv')
```

## Evaluation signals

- Each output CSV has exactly 2,736 feature columns (plus index column) and float dtype throughout
- Column means of the standardized matrix are ~0 and standard deviations ~1 per feature
- All split files (train/val/test) share an identical ordered 2,736-column header so encoder weights and latents map to the same features
- pandas.read_csv(..., index_col=0) loads each file without errors and row counts match the intended split assignment
- y_* label files (if written) have the same number of rows and an aligned index with the corresponding X_* file

## Limitations

- The 2,736-feature set and resulting pretrained model are specific to HILIC positive mode; other LC modes and modalities are not covered
- The matrix is limited to MS1 ion features — mzLearn does not perform de-isotoping or MS/MS identification, so no MS/MS columns can be included
- No batch covariates or domain-invariant regularization are built into the feature matrix; large-scale deployments may need to add these
- Feature matrices depend on mzLearn's drift correction; public repositories often lack standardized QC samples to estimate instrument drift, which can affect the robust peak set

## Evidence

- [readme] Output: A two-dimensional table of detected features defined by median rt and m/z values, with normalized intensities across samples.: [evidence span withheld — anchor status: not_found]
- [readme] data_location: str — folder containing X_Pretrain_* (float; z-scored; [n, 2736]) and optional y_Pretrain_*.: [evidence span withheld — anchor status: not_found]
- [readme] Feature matrices: CSV of floats, shape [n_samples, 2736] (the robust mzLearn peak set).: [evidence span withheld — anchor status: not_found]
