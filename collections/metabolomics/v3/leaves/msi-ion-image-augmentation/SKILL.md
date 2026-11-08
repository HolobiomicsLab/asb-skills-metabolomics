---
name: msi-ion-image-augmentation
description: Use when training a contrastive encoder on MSI ion images and you need
  augmentation pairs that reflect MSI priors — Poisson ion counts, missing values,
  or isotope-ion relationships — rather than generic image transforms.
when_to_use_negative:
- When the goal is identifying colocalized ions but the augmentation objective is
  not specifically isotope-ion contrastive training — the intensity-dependent missing-value
  step belongs to T_ISO only, so applying the full T_ISO pipeline in COL mode would
  contaminate the training distribution.
- When augmenting natural images (e.g. for standard SimSiam on photographs) — MSI-specific
  transforms like Poisson noise and missing-value masking are inappropriate there.
- When the input is already an embedding or feature table (e.g. after UMAP reduction
  to 20 dimensions) — augmentation must be applied to raw ion images before encoder
  training.
edam_operation: http://edamontology.org/operation_3674
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3382
tools:
- name: DeepION
  role: Model whose Data Augmentation module T_COL/T_ISO this skill implements; provides
    the COL and ISO training modes that consume the augmented views
  repo: https://github.com/BioNet-XMU/DeepION
- name: PyTorch
  role: Tensor framework for implementing the augmentation transformations (jitter,
    Poisson noise, missing masks) as differentiable operations
- name: kornia
  role: PyTorch image-operator library used to implement color-jitter and filtering
    transforms on ion images
- name: SimSiam
  role: Contrastive architecture that trains on the generated augmented view pairs
    (x1, x2)
- name: Python
  role: Implementation language for the augmentation module (e.g. col_augmentation.py,
    iso_augmentation.py)
provenance:
  source_task_ids:
  - task_002
  - task_008
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:d7336d5c522522626dc4a1808e3005dec26e9abf9baf498657133ee517f12a38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioNet-XMU/DeepION
  tools:
  - DeepION
  - Nvidia GTX 2080Ti
  - PyTorch
  - Python
  - Resnet18
  - SimSiam
  - kornia
  edam_operation: http://edamontology.org/operation_3674
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# msi-ion-image-augmentation

## Summary

Applies DeepION's modality-specific augmentation pipelines T_COL and T_ISO to ion images from mass spectrometry imaging data, generating augmented views for contrastive training by simulating MSI characteristics (Poisson-distributed ion abundance, random and intensity-dependent missing patterns) rather than natural-image transforms alone.

## When to use

Use when training a contrastive encoder on MSI ion images and you need augmentation pairs that reflect MSI priors — Poisson ion counts, missing values, or isotope-ion relationships — rather than generic image transforms.

## When NOT to use

- When the goal is identifying colocalized ions but the augmentation objective is not specifically isotope-ion contrastive training — the intensity-dependent missing-value step belongs to T_ISO only, so applying the full T_ISO pipeline in COL mode would contaminate the training distribution.
- When augmenting natural images (e.g. for standard SimSiam on photographs) — MSI-specific transforms like Poisson noise and missing-value masking are inappropriate there.
- When the input is already an embedding or feature table (e.g. after UMAP reduction to 20 dimensions) — augmentation must be applied to raw ion images before encoder training.

## Inputs

- Preprocessed MSI ion-image matrix with two-dimensional shape [X*Y, P] (e.g. DATASET/Pos_brain_data_matrix.txt, spatial shape 198×422 pixels × P ions)
- Individual ion image (2-D array extracted for augmentation testing)

## Outputs

- Two augmented ion-image views of identical spatial dimensions to the input, serving as positive pairs for contrastive training
- Augmented views saved as tensors/arrays (e.g. augmented_views.npy) with simulated missing entries

## How to apply

Compose a single T_COL pipeline of four transformations — color jitter and filtering (to reduce sensitivity to image brightness and focus on critical features), Poisson noise (to emulate the Poisson distribution of ion abundance in MSI), and random missing value (to emulate the random missing pattern of MSI) — built on PyTorch/kornia tensor operations, applying them to each [X*Y, P] ion image extracted from a preprocessed matrix such as DATASET/Pos_brain_data_matrix.txt (spatial shape 198×422 pixels). Generate two augmented views (x1, x2) of the same ion image to serve as positive pairs for SimSiam-style contrastive training. When augmenting for isotope-ion discovery (ISO mode), extend the pipeline with an additional intensity-dependent missing-value operation in which the missing ratio of each ion is set proportional to its intensity. Do not include the intensity-dependent step in T_COL — it is exclusive to ISO mode. Both views must have identical spatial dimensions to the input and must differ numerically while preserving the underlying spatial structure, so downstream encoder features can be reduced to 20 dimensions with UMAP and min–max scaled to [0,1] for distance-based retrieval.

## Related tools

- **DeepION** (Model whose Data Augmentation module T_COL/T_ISO this skill implements; provides the COL and ISO training modes that consume the augmented views) — https://github.com/BioNet-XMU/DeepION
- **PyTorch** (Tensor framework for implementing the augmentation transformations (jitter, Poisson noise, missing masks) as differentiable operations)
- **kornia** (PyTorch image-operator library used to implement color-jitter and filtering transforms on ion images)
- **SimSiam** (Contrastive architecture that trains on the generated augmented view pairs (x1, x2))
- **Python** (Implementation language for the augmentation module (e.g. col_augmentation.py, iso_augmentation.py))

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- Augmented views must match the input ion-image shape exactly (e.g. 198×422 for the rat brain positive-mode dataset) with no dimension mismatch
- Each augmented view must be numerically distinct from the original ion image while retaining the same dimensionality
- Simulated missing entries must appear in the augmented tensors, and in the ISO-mode output their frequency must qualitatively increase with ion intensity (intensity-dependent check)
- Random missing-value and Poisson-noise transforms must be present in T_COL; the intensity-dependent missing-value transform must appear only when the pipeline is used for ISO mode
- The underlying spatial structure of the ion image must remain recognizable after augmentation, so encoder features remain suitable for UMAP reduction to 20 dimensions and downstream similarity retrieval

## Limitations

- The exact missing-ratio proportionality constant for the intensity-dependent missing-value operation is not stated numerically in the article or README, requiring qualitative rather than quantitative validation of the ISO transform.
- The transforms assume the input matrix is already preprocessed into [X*Y, P] form with an explicit spatial shape (198 422 in the run example); raw instrument output must first be converted.
- Augmentation batch size during training is constrained by GPU memory (article reports training on a single Nvidia GTX 2080Ti), which may limit the number of augmented pairs generated per iteration.
- The filtering and color-jitter choices are justified for ion images to reduce brightness sensitivity, but their transferability to other MSI modalities (e.g. negative ion mode or different tissue types) is not validated in the article.

## Evidence

- [methods] color jitter and filtering (to reduce brightness sensitivity and focus on critical features), Poisson noise (to emulate the Poisson distribution of ion abundance in MSI), and random missing value: "including color jitter, filtering, Poisson noise, random missing value, and intensity-dependent missing value"
- [methods] T_ISO extends the T_COL augmentation pipeline with an additional intensity-dependent missing-value step: "T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode, while T_ISO introduces an additional process of intensity-dependent missing value in ISO mode"
- [readme] The augmented views must be of identical spatial dimensions to the input and numerically distinct, preserving spatial structure for downstream retrieval: "The original ion image is first imported into the data augmentation module "T"  to generate two augmented images"
- [results] DeepION uses MSI-specific augmentation that outperforms SimSiam's natural-image augmentation by ~25 percentage points on the colocalized-ion benchmark: "the COL mode of the DeepION model achieves 86.73%, which outperforms the second-best SimSiam model by about 25%"
