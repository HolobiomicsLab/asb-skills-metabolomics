---
name: pytorch-image-transform-implementation
description: Use when implementing data augmentation for ion images in mass spectrometry
  imaging, e.g., producing augmented view pairs for self-supervised contrastive training
  of an encoder like DeepION's ResNet18 backbone.
when_to_use_negative:
- The task requires ISO-mode augmentation — the intensity-dependent missing value
  transform is exclusive to T_ISO and must not be included in T_COL
- Input is already an augmented or representation vector (e.g., the 512-dim encoder
  output or 20-dim reduced vector); transforms apply to raw ion images only
- The data are natural images rather than MSI ion images — Poisson-noise and random-missing
  transforms are motivated by MSI-specific characteristics (Poisson ion abundance,
  random missing pattern)
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: PyTorch
  role: Framework in which the augmentation transforms are implemented
- name: kornia
  role: Provides image operators (version 0.4.1) used to realize the color-jitter
    and filtering transforms on ion images
  repo: https://github.com/gankLei-X/DeepION
- name: DeepION
  role: Reference model whose Data Augmentation module T defines T_COL (and T_ISO)
    for generating augmented image pairs for contrastive training
  repo: https://github.com/gankLei-X/DeepION
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:db9f989d2dbdf197260f07c4a44e4e9e46ee1089f586c354b0285e6d2b3f8f4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gankLei-X/DeepION
  tools:
  - DeepION
  - PyTorch
  - Python
  - kornia
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# pytorch-image-transform-implementation

## Summary

Implement MSI-specific image augmentation transforms in PyTorch (using kornia operators) that generate augmented ion-image pairs for contrastive training, as in DeepION's Data Augmentation module T_COL: color jitter, filtering, Poisson noise, and random missing value.

## When to use

Use when implementing data augmentation for ion images in mass spectrometry imaging, e.g., producing augmented view pairs for self-supervised contrastive training of an encoder like DeepION's ResNet18 backbone.

## When NOT to use

- The task requires ISO-mode augmentation — the intensity-dependent missing value transform is exclusive to T_ISO and must not be included in T_COL
- Input is already an augmented or representation vector (e.g., the 512-dim encoder output or 20-dim reduced vector); transforms apply to raw ion images only
- The data are natural images rather than MSI ion images — Poisson-noise and random-missing transforms are motivated by MSI-specific characteristics (Poisson ion abundance, random missing pattern)

## Inputs

- Preprocessed MSI matrix data with two-dimensional shape [X*Y,P] (e.g., DATASET/Pos_brain_data_matrix.txt; 198 x 422 pixels for the rat brain section)
- Preprocessed MSI peak list with one-dimensional shape [P,1] (e.g., DATASET/Pos_brain_data_peak.csv)
- A single ion image extracted from the matrix, reshaped to the stated spatial shape (198 x 422)

## Outputs

- Two augmented views of the ion image with identical spatial dimensions (198 x 422), saved as augmented_views.npy
- Reusable PyTorch/kornia transform functions implementing color jitter, filtering, Poisson noise, and random missing value (e.g., col_augmentation.py)

## How to apply

Load the preprocessed MSI matrix (e.g., DATASET/Pos_brain_data_matrix.txt with spatial shape 198 x 422 pixels x P ions) and extract a single ion image. Implement four T_COL transforms in PyTorch with kornia 0.4.1: color jitter and filtering (borrowed from natural-image augmentation so the encoder is insensitive to brightness and focuses on critical features), Poisson noise (perturbing ion abundance per a Poisson distribution, reflecting the count-based nature of MS ion detection), and random missing value (stochastic pixel masking simulating MSI's random missing pattern). Apply the four transforms to produce two augmented views of identical spatial dimensions; note the intensity-dependent missing-value transform belongs only to T_ISO and must be excluded from T_COL.

## Related tools

- **PyTorch** (Framework in which the augmentation transforms are implemented)
- **kornia** (Provides image operators (version 0.4.1) used to realize the color-jitter and filtering transforms on ion images) — https://github.com/gankLei-X/DeepION
- **DeepION** (Reference model whose Data Augmentation module T defines T_COL (and T_ISO) for generating augmented image pairs for contrastive training) — https://github.com/gankLei-X/DeepION

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- The script runs without error and col_augmentation.py exists
- Each saved augmented view matches the input ion-image spatial shape (198 x 422)
- Each augmented view is numerically distinct from the original ion image while retaining identical dimensionality
- The transform set contains exactly color jitter, filtering, Poisson noise, and random missing value — no intensity-dependent missing value (T_ISO-only) transform
- Expert review confirms transform fidelity against DeepION Section 3.1

## Limitations

- The intensity-dependent missing-value transform must NOT be applied in COL mode; it is exclusive to ISO mode
- Transforms are validated only on the 198 x 422 rat brain positive-mode MSI data described in the workflow; fidelity for other shapes/pixel counts is untested here
- The rationale (brightness insensitivity, Poisson count noise, random missing pattern) is specific to MSI data characteristics and may not transfer to other imaging modalities

## Evidence

- [methods] color jitter, filtering, Poisson noise, random missing value, and intensity-dependent missing value: "including color jitter, filtering, Poisson noise, random missing value, and intensity-dependent missing value"
- [abstract] a unique data augmentation strategy is designed by considering the characteristics of MSI data, such as the Poisson distribution of ion abundance and a random pattern of missing values: "a unique data augmentation strategy is designed by considering the characteristics of MSI data, such as the Poisson distribution of ion abundance and a random pattern of missing values"
- [readme] the T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode: "the T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode, while T_ISO introduces an additional process of intensity-dependent missing value in ISO"
- [methods] The DeepMSI model is implemented in Python using the PyTorch library: "The DeepMSI model is implemented in Python using the PyTorch library"
