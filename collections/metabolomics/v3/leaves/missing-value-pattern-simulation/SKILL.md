---
name: missing-value-pattern-simulation
description: Use when augmenting MSI ion images for contrastive/self-supervised training
  and you need augmented views that mimic the missing-value artifacts of MS ion detection,
  as in DeepION's T_COL or T_ISO transforms.
when_to_use_negative:
- 'Augmenting for the ISO mode of DeepION: the intensity-dependent missing-value transform
  must be added there, so the COL-only recipe is the wrong choice'
- Input is already an augmented view or a downstream representation vector (e.g. the
  20-dimensional UMAP output) — augmentation applies only to original ion images before
  encoding
- The task is standard natural-image contrastive learning where MSI-specific artifacts
  (Poisson noise, missing values) are not present in the data modality
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: DeepION
  role: Source framework whose T_COL/T_ISO augmentation modules define the random
    missing-value and intensity-dependent missing-value transforms
  repo: https://github.com/gankLei-X/DeepION
- name: PyTorch
  role: Library used to implement the DeepMSI model and its augmentation transforms
- name: kornia
  role: Differentiable image operator library (v0.4.1) used for image transforms such
    as color jitter and filtering alongside the missing-value masking
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:b4e08e663b30b96c6151d2eb0f6406a9552a7feb0b766e2d6a95f2fed71e8f7c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gankLei-X/DeepION
  tools:
  - DeepION
  - PyTorch
  - Python
  - kornia
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# missing-value-pattern-simulation

## Summary

Stochastically masks pixel values in ion images to simulate the random missing-value pattern characteristic of mass spectrometry imaging (MSI) data, used as part of DeepION's T_COL data augmentation (with an intensity-dependent variant reserved for T_ISO).

## When to use

Use when augmenting MSI ion images for contrastive/self-supervised training and you need augmented views that mimic the missing-value artifacts of MS ion detection, as in DeepION's T_COL or T_ISO transforms.

## When NOT to use

- Augmenting for the ISO mode of DeepION: the intensity-dependent missing-value transform must be added there, so the COL-only recipe is the wrong choice
- Input is already an augmented view or a downstream representation vector (e.g. the 20-dimensional UMAP output) — augmentation applies only to original ion images before encoding
- The task is standard natural-image contrastive learning where MSI-specific artifacts (Poisson noise, missing values) are not present in the data modality

## Inputs

- Preprocessed MSI data matrix, 2D shape [X*Y, P] (e.g. DATASET/Pos_brain_data_matrix.txt), reshaped per-ion to spatial images (198 x 422 for the rat brain positive-mode dataset)
- Single ion image extracted from the matrix

## Outputs

- Two augmented ion-image views with missing-value masks applied, of the same spatial shape as the input (e.g. 198 x 422), suitable for saving as augmented_views.npy and feeding to the ResNet18-based encoder

## How to apply

Load a preprocessed MSI matrix (e.g. DATASET/Pos_brain_data_matrix.txt, spatial shape 198 x 422 pixels x P ions) and reshape a single ion image to its spatial dimensions. Apply the random missing-value transform by stochastically masking pixel values, optionally in combination with color jitter, filtering, and Poisson noise (implemented in PyTorch with kornia 0.4.1 image operators) to generate two augmented views of identical spatial dimensions. Rationale: MSI data exhibit Poisson-distributed ion abundance and a random pattern of missing values, so augmentations encoding these characteristics teach the encoder to be robust to MS-specific artifacts. Critically, only the random missing-value transform belongs in T_COL; the intensity-dependent missing-value transform is exclusive to the ISO mode (isotope-ion discovery) and must not be included when augmenting for colocalized-ion training.

## Related tools

- **DeepION** (Source framework whose T_COL/T_ISO augmentation modules define the random missing-value and intensity-dependent missing-value transforms) — https://github.com/gankLei-X/DeepION
- **PyTorch** (Library used to implement the DeepMSI model and its augmentation transforms)
- **kornia** (Differentiable image operator library (v0.4.1) used for image transforms such as color jitter and filtering alongside the missing-value masking)

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- Each augmented view is numerically distinct from the original ion image while retaining identical dimensionality (198 x 422 for the rat brain dataset)
- The two generated views share the same spatial dimensions as the input ion image, satisfying SimSiam-style twin-encoder input requirements
- T_COL output contains stochastic missing-value masks but no intensity-dependent missing-value component (that transform appears only in T_ISO)
- Script runs without error and saves views (e.g. augmented_views.npy); expert review confirms transform fidelity against Section 3.1 of the paper

## Limitations

- The intensity-dependent missing-value transform is exclusive to ISO mode and must not be mixed into COL-mode augmentation
- The augmentation is tailored to MSI characteristics (Poisson-distributed abundance, random missing pattern); it presumes preprocessed matrix input with known spatial shape supplied via --input_shape
- The paper's discussion section does not report limitations or ablations specific to the missing-value transform itself

## Evidence

- [readme] T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode, while T_ISO introduces an additional process of intensity-dependent missing value in ISO mode: "the T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode, while T_ISO introduces an additional process of intensity-dependent missing value in ISO"
- [abstract] a unique data augmentation strategy is designed by considering the characteristics of MSI data, such as the Poisson distribution of ion abundance and a random pattern of missing values: "a unique data augmentation strategy is designed by considering the characteristics of MSI data, such as the Poisson distribution of ion abundance and a random pattern of missing values"
- [methods] including color jitter, filtering, Poisson noise, random missing value, and intensity-dependent missing value: "including color jitter, filtering, Poisson noise, random missing value, and intensity-dependent missing value"
- [readme] The original ion image is first imported into the data augmentation module "T"  to generate two augmented images: "The original ion image is first imported into the data augmentation module "T"  to generate two augmented images"
