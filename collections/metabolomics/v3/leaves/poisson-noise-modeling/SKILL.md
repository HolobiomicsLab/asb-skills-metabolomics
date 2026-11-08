---
name: poisson-noise-modeling
description: Use when generating augmented views of MSI ion images for self-supervised/contrastive
  training and you need the augmentation to reflect the Poisson distribution of ion
  counts; also relevant when simulating count-noise on ion images.
when_to_use_negative:
- The input is not MSI ion-image data with count-like (non-negative integer-scale)
  abundances — Poisson noise assumes count-based ion detection.
- You are building the ISO-mode T_ISO pipeline and need the intensity-dependent missing-value
  step — that operation is additive to, not a replacement for, Poisson noise and belongs
  to a different modality.
- The goal is general natural-image augmentation without MSI-specific priors — plain
  color jitter/filtering suffices and Poisson noise would be unmotivated.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: DeepION
  role: Framework whose COL-mode data augmentation module T_COL embeds the Poisson-noise
    transform to generate augmented ion-image pairs
  repo: https://github.com/gankLei-X/DeepION
- name: PyTorch
  role: Tensor backend used to implement the Poisson-noise perturbation of ion abundance
- name: kornia
  role: Image-operator library (pinned at 0.4.1) used for the companion color-jitter
    and filtering transforms in the T_COL pipeline
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:eb8df6ed1bfd0f58f47aace130e6ab85cc3577f6937e2e92a87890c7ac7b2373
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gankLei-X/DeepION
  tools:
  - DeepION
  - PyTorch
  - Python
  - kornia
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# poisson-noise-modeling

## Summary

Applies Poisson-distributed noise to ion images from mass spectrometry imaging (MSI) data, as used in DeepION's data augmentation module to emulate the count-based, Poisson-distributed nature of ion abundance detection before contrastive training.

## When to use

Use when generating augmented views of MSI ion images for self-supervised/contrastive training and you need the augmentation to reflect the Poisson distribution of ion counts; also relevant when simulating count-noise on ion images.

## When NOT to use

- The input is not MSI ion-image data with count-like (non-negative integer-scale) abundances — Poisson noise assumes count-based ion detection.
- You are building the ISO-mode T_ISO pipeline and need the intensity-dependent missing-value step — that operation is additive to, not a replacement for, Poisson noise and belongs to a different modality.
- The goal is general natural-image augmentation without MSI-specific priors — plain color jitter/filtering suffices and Poisson noise would be unmotivated.

## Inputs

- Preprocessed MSI data matrix, 2-D shape [X*Y, P] (e.g. DATASET/Pos_brain_data_matrix.txt)
- Single ion image extracted from the matrix, spatial shape 198 x 422 pixels for the rat brain positive-mode dataset
- PyTorch tensor of the ion image with non-negative abundance values

## Outputs

- Poisson-noised ion image(s) (tensors or .npy arrays, e.g. augmented_views.npy) with the same spatial dimensions as the input
- Augmented view pairs (x1, x2) usable as positive pairs for SimSiam-style contrastive training in COL mode

## How to apply

Load a single ion image from the preprocessed MSI matrix (2-D array [X*Y, P], e.g. DATASET/Pos_brain_data_matrix.txt reshaped to the spatial shape 198 x 422 for the rat brain dataset) as a PyTorch tensor. Perturb the ion abundance values according to a Poisson distribution — e.g. drawing each pixel's noisy value from Poisson(intensity) — so that noise magnitude is tied to the ion counts, mirroring shot noise in MS ion detection. Compose this Poisson-noise transform with the other T_COL transforms (color jitter and filtering via kornia 0.4.1 image operators, and random missing value) to produce augmented views for SimSiam-style contrastive training. Do NOT include the intensity-dependent missing-value operation, which is exclusive to the ISO-mode T_ISO pipeline. Rationale: DeepION's augmentation is designed 'by considering the characteristics of MSI data, such as the Poisson distribution of ion abundance and a random pattern of missing values'.

## Related tools

- **DeepION** (Framework whose COL-mode data augmentation module T_COL embeds the Poisson-noise transform to generate augmented ion-image pairs) — https://github.com/gankLei-X/DeepION
- **PyTorch** (Tensor backend used to implement the Poisson-noise perturbation of ion abundance)
- **kornia** (Image-operator library (pinned at 0.4.1) used for the companion color-jitter and filtering transforms in the T_COL pipeline)

## Examples

```
import torch; x = torch.load('ion_image.pt').clamp(min=0); x_noisy = torch.poisson(x)
```

## Evaluation signals

- The Poisson-noised output preserves the input ion-image spatial shape (198 x 422 for the rat brain test case).
- The augmented view is numerically distinct from the original ion image while retaining the same dimensionality.
- Noise magnitude scales with pixel intensity: low-abundance pixels show small perturbations, high-abundance pixels larger absolute fluctuation (Poisson property).
- Output values remain non-negative and count-like, consistent with ion-count data.
- The underlying spatial structure of the ion image is visually preserved (qualitative fidelity check against Section 3.1).

## Limitations

- Poisson noise only makes sense on non-negative abundance values; applying it after arbitrary rescaling requires care to keep values count-like.
- Environment pins are strict (python 3.5–3.7, pytorch 1.8.2, kornia 0.4.1, numpy >= 1.8.0, umap 0.5.1); newer versions may break the image operators.
- The intensity-dependent missing-value transform is exclusive to ISO mode and must NOT be included when composing the COL-mode T_COL pipeline.
- The article reports no stated limitations or weaknesses of DeepION in its discussion section, so robustness of the noise model outside rat brain MSI data is uncharacterized.

## Evidence

- [abstract] a unique data augmentation strategy is designed by considering the characteristics of MSI data, such as the Poisson distribution of ion abundance and a random pattern of missing values: "a unique data augmentation strategy is designed by considering the characteristics of MSI data, such as the Poisson distribution of ion abundance and a random pattern of missing values"
- [intro] we propose two data augmentation strategies especially for the COL and ISO mods by considering the characteristics of the Poisson distribution and missing patterns in MSI.: "we propose two data augmentation strategies especially for the COL and ISO mods by considering the characteristics of the Poisson distribution and missing patterns in MSI."
- [methods] including color jitter, filtering, Poisson noise, random missing value, and intensity-dependent missing value: "including color jitter, filtering, Poisson noise, random missing value, and intensity-dependent missing value"
- [readme] the T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode, while T_ISO introduces an additional process of intensity-dependent missing value in ISO mode.: "the T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode, while T_ISO introduces an additional process of intensity-dependent missing value in ISO"
