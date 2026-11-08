---
name: contrastive-view-generation
description: Use when implementing self-supervised contrastive learning on mass spectrometry
  imaging ion images and you need modality-specific augmented positive pairs that
  respect MSI characteristics (Poisson-distributed ion abundance and missing-value
  patterns).
when_to_use_negative:
- Input is not an ion image from mass spectrometry imaging (e.g. general natural images)
  — the Poisson-noise and missing-value transforms are grounded in MSI-specific abundance
  and missingness characteristics
- Building the ISO pipeline for a colocalized-ion (COL) task — the intensity-dependent
  missing-value transform belongs only to T_ISO and would incorrectly bias COL-mode
  training
- You need labeled supervised augmentation for classification rather than unlabeled
  positive-pair generation for self-supervised contrastive learning
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: DeepION
  role: Framework whose COL and ISO modes consume the augmented view pairs for contrastive
    ion-image representation learning
  repo: https://github.com/BioNet-XMU/DeepION
- name: kornia
  role: Provides the image operators (version 0.4.1) used to implement the color-jitter
    and filtering transforms on ion images
- name: PyTorch
  role: Tensor backend (version 1.8.2) in which the augmentation transforms and positive-pair
    pipeline are implemented
- name: SimSiam
  role: Contrastive architecture the augmented (x1, x2) pairs feed into; twin ResNet18
    encoders with stop-gradient prevent collapse
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:d975bee0a84633d590e3e6fe66faf84ce00b172f1befaeeb2a31c2b78f242816
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioNet-XMU/DeepION
  tools:
  - DeepION
  - PyTorch
  - Python
  - SimSiam
  - kornia
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# contrastive-view-generation

## Summary

MSI-specific data augmentation that generates two augmented views of a single ion image to serve as positive pairs for SimSiam-based contrastive training in DeepION. The COL variant (T_COL) applies color jitter, filtering, Poisson noise, and random missing value; the ISO variant (T_ISO) adds intensity-dependent missing value.

## When to use

Use when implementing self-supervised contrastive learning on mass spectrometry imaging ion images and you need modality-specific augmented positive pairs that respect MSI characteristics (Poisson-distributed ion abundance and missing-value patterns).

## When NOT to use

- Input is not an ion image from mass spectrometry imaging (e.g. general natural images) — the Poisson-noise and missing-value transforms are grounded in MSI-specific abundance and missingness characteristics
- Building the ISO pipeline for a colocalized-ion (COL) task — the intensity-dependent missing-value transform belongs only to T_ISO and would incorrectly bias COL-mode training
- You need labeled supervised augmentation for classification rather than unlabeled positive-pair generation for self-supervised contrastive learning

## Inputs

- Preprocessed MSI data matrix as 2-D array [X*Y, P] (e.g. DATASET/Pos_brain_data_matrix.txt)
- MSI input spatial shape (e.g. 198 x 422 pixels)
- Selected mode flag: 'COL' or 'ISO' (determines whether T_COL or T_ISO is applied)

## Outputs

- Two augmented ion-image views (x1, x2) as PyTorch tensors with the original spatial shape, for use as positive pairs in SimSiam-style contrastive training
- Python augmentation module (e.g. col_augmentation.py / iso_augmentation.py) invocable from the DeepION training loop

## How to apply

Load the preprocessed MSI matrix (2-D shape [X*Y, P], e.g. 198 x 422 pixels x P ions from DATASET/Pos_brain_data_matrix.txt) and reshape each ion image to its spatial dimensions. Compose T_COL from four transforms: color jitter and filtering (via kornia 0.4.1 image operators) to make the encoder insensitive to brightness and focus on critical features; Poisson noise to emulate the Poisson distribution of ion abundance; and random missing value to simulate the random missing pattern of MSI data. For isotope-ion contrastive training, extend to T_ISO by additively applying an intensity-dependent missing-value operation in which each ion's missing ratio is set proportional to its intensity — this transform is exclusive to ISO mode and must NOT be included in T_COL. Apply the pipeline to a single ion image to return two augmented views (x1, x2) of identical spatial shape as PyTorch tensors, suitable as positive pairs for the ResNet18-based shared-encoder SimSiam training loop with contrastive loss and stop-gradient.

## Related tools

- **DeepION** (Framework whose COL and ISO modes consume the augmented view pairs for contrastive ion-image representation learning) — https://github.com/BioNet-XMU/DeepION
- **kornia** (Provides the image operators (version 0.4.1) used to implement the color-jitter and filtering transforms on ion images)
- **PyTorch** (Tensor backend (version 1.8.2) in which the augmentation transforms and positive-pair pipeline are implemented)
- **SimSiam** (Contrastive architecture the augmented (x1, x2) pairs feed into; twin ResNet18 encoders with stop-gradient prevent collapse)

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- Each augmented view retains the input ion-image spatial shape (e.g. 198 x 422) while being numerically distinct from the original
- T_COL output contains exactly the four COL transforms (color jitter, filtering, Poisson noise, random missing) with no intensity-dependent missing value
- T_ISO output exhibits simulated missing entries whose frequency increases with ion intensity (qualitative intensity-dependence check)
- Augmented views preserve the underlying spatial structure of the original ion image
- Pipeline runs without error as a Python module and returns well-formed tensors accepted by the SimSiam training loop

## Limitations

- The intensity-dependent missing-value transform is exclusive to ISO mode and must not be applied in COL mode
- Requires kornia 0.4.1 and PyTorch 1.8.2 per the README environment; kornia version mismatches may break the color-jitter/filtering operators
- Transforms are designed for ion images from MSI data with Poisson-distributed abundance and missing patterns; their rationale does not transfer to generic image data

## Evidence

- [methods] T_COL augmentation rationale: "including color jitter, filtering, Poisson noise, random missing value, and intensity-dependent missing value"
- [abstract] MSI augmentation rationale: "a unique data augmentation strategy is designed by considering the characteristics of MSI data, such as the Poisson distribution of ion abundance and a random pattern of missing values"
- [readme] COL vs ISO transforms: "the T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode, while T_ISO introduces an additional process of intensity-dependent missing value in ISO"
- [readme] input data format: "The preprocessed MSI matrix data with two-dimensional shape [X*Y,P], where X and Y represent the pixel numbers of horizontal and vertical coordinates of MSI data, and P represents the number of ions"
- [methods] SimSiam basis: "The proposed DeepION model is constructed based on the SimSiam model"
