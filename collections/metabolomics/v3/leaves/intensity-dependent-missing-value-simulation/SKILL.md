---
name: intensity-dependent-missing-value-simulation
description: Use when preparing augmented ion-image pairs from a preprocessed MSI
  matrix ([X*Y,P]) for DeepION ISO-mode isotope-ion representation training and the
  missing values must depend on ion intensity.
when_to_use_negative:
- 'The goal is COL-mode colocalized-ion augmentation: T_COL uses only color jitter,
  filtering, Poisson noise and random missing value and must not include the intensity-dependent
  missing-value step.'
- The input is raw or unpreprocessed MSI data rather than the preprocessed 2-D matrix
  [X*Y,P] and 1-D peak list [P,1] the pipeline expects.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: DeepION
  role: 'Host framework: its T_ISO data-augmentation module generates the augmented
    image pairs; augmentation is composed as a Python module invoked by the ISO-mode
    training loop'
  repo: https://github.com/BioNet-XMU/DeepION
- name: PyTorch
  role: Implements the color jitter, filtering, Poisson noise, random missing value,
    and intensity-dependent missing-value operations as tensor-based transforms
- name: Python
  role: Language in which the iso_augmentation.py module is written and run
- name: kornia
  role: Acknowledged dependency package whose utilities support the augmentation operations
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:5af66c8346ccc26ef9b5b7502c3cf5e21e83e94374e4df2fa31f87a135ac65d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioNet-XMU/DeepION
  tools:
  - DeepION
  - PyTorch
  - Python
  - Resnet18
  - kornia
  - rMSI
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# intensity-dependent-missing-value-simulation

## Summary

Applies the DeepION T_ISO augmentation: adds an intensity-dependent missing-value step on top of the T_COL pipeline (color jitter, filtering, Poisson noise, random missing value) so augmented ion-image pairs emulate MSI's missing patterns and can serve as positive pairs for SimSiam-style ISO-mode contrastive training.

## When to use

Use when preparing augmented ion-image pairs from a preprocessed MSI matrix ([X*Y,P]) for DeepION ISO-mode isotope-ion representation training and the missing values must depend on ion intensity.

## When NOT to use

- The goal is COL-mode colocalized-ion augmentation: T_COL uses only color jitter, filtering, Poisson noise and random missing value and must not include the intensity-dependent missing-value step.
- The input is raw or unpreprocessed MSI data rather than the preprocessed 2-D matrix [X*Y,P] and 1-D peak list [P,1] the pipeline expects.

## Inputs

- Preprocessed MSI matrix data with 2-D shape [X*Y,P] (e.g. DATASET/Pos_brain_data_matrix.txt, rat brain positive-mode 198 x 422 pixels)
- Preprocessed MSI peak list with 1-D shape [P,1] (e.g. Pos_brain_data_peak.csv)

## Outputs

- Two augmented ion-image views (x1, x2) per input ion image, well-formed tensors of the original shape, suitable as positive pairs for ISO-mode SimSiam contrastive training
- Simulated missing entries whose frequency increases with ion intensity

## How to apply

Load the preprocessed MSI ion-image matrix, e.g. a 2-D array [X*Y,P] such as DATASET/Pos_brain_data_matrix.txt, into PyTorch tensors. First run the base T_COL transforms as callables: color jitter and filtering (reducing brightness sensitivity and focusing on critical features), Poisson noise (emulating the Poisson distribution of ion abundance in MSI), and random missing value (emulating random missing patterns in MSI). Then apply the ISO-specific extension additively: an intensity-dependent missing-value operation in which the missing ratio of each ion is set proportional to its intensity. Compose these into a T_ISO pipeline that, given one ion image, returns two augmented views (x1,x2) usable as positive pairs for SimSiam with a pretrained Resnet18 backbone in ISO mode. Expose it as a Python module (iso_augmentation.py) on PyTorch tensor operations so the DeepION run.py --mode ISO training loop can invoke it.

## Related tools

- **DeepION** (Host framework: its T_ISO data-augmentation module generates the augmented image pairs; augmentation is composed as a Python module invoked by the ISO-mode training loop) — https://github.com/BioNet-XMU/DeepION
- **PyTorch** (Implements the color jitter, filtering, Poisson noise, random missing value, and intensity-dependent missing-value operations as tensor-based transforms)
- **Python** (Language in which the iso_augmentation.py module is written and run)
- **kornia** (Acknowledged dependency package whose utilities support the augmentation operations)

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode ISO --ion_mode positive --output_file Pos_ISO_result
```

## Evaluation signals

- Augmented outputs are well-formed tensors of the original ion-image shape.
- Simulated missing entries show qualitatively increasing frequency with ion intensity (intensity-dependence check).
- The augmentation preserves the underlying spatial structure of the ion image.
- The pipeline returns two augmented views (x1, x2) per input image suitable as positive pairs for SimSiam ISO-mode training.

## Limitations

- The intensity-dependent missing-value step is ISO-mode specific: it must be omitted for COL-mode runs (T_COL includes only color jitter, filtering, Poisson noise, and random missing value).
- Requires preprocessed MSI matrix [X*Y,P] and peak list [P,1] inputs; running DeepION itself needs specific pinned versions (pytorch 1.8.2, kornia 0.4.1, umap 0.5.1, torchvision 0.11.3, python 3.5/3.6/3.7).

## Evidence

- [other] T_ISO extends the T_COL augmentation pipeline (color jitter, filtering, Poisson noise, random missing value) with an additional intensity-dependent missing-value step: [evidence span withheld — anchor status: not_found]
- [other] ISO-specific extension: intensity-dependent missing value: [evidence span withheld — anchor status: not_found]
- [readme] the T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode, while T_ISO introduces an additional process of intensity-dependent missing value in ISO mode: "the T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode, while T_ISO introduces an additional process of intensity-dependent missing value in ISO"
- [other] run DeepION in ISO mode: [evidence span withheld — anchor status: not_found]
- [abstract] unique data augmentation strategy considering MSI characteristics: "a unique data augmentation strategy is designed by considering the characteristics of MSI data, such as the Poisson distribution of ion abundance and a random pattern of missing values"
