---
name: simsiam-based-self-supervised-pipeline-integration
description: Use when you need low-dimensional, annotation-free representations of
  ion images from an MSI matrix for downstream colocalized-ion or isotope-ion identification
  tasks.
when_to_use_negative:
- Data are raw, unpreprocessed MSI spectra rather than a preprocessed [X*Y, P] matrix
  and a [P,1] peak list — the pipeline expects preprocessed matrix/peak inputs.
- Labels or annotated pairs are already available and a supervised model is preferred
  — DeepION is designed for self-supervised representation without manual annotation.
- The task is general image representation outside MSI, where MSI-specific augmentation
  (Poisson noise, random/intensity-dependent missing value) is not appropriate.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: DeepION
  role: End-to-end implementation of the SimSiam-based contrastive pipeline for ion-image
    representation in MSI
  repo: https://github.com/gankLei-X/DeepION
- name: SimSiam
  role: Base architecture on which DeepION is constructed (twin networks, contrastive
    loss with stop-gradient)
- name: Resnet18
  role: Pretrained backbone network for the twin encoders
- name: PyTorch
  role: Python library in which the model is implemented
- name: UMAP
  role: Dimensionality reduction of the 512-d representation to the 20-d output vector
- name: kornia
  role: Dependency used by DeepION (acknowledged package)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:758920de03aa032e0b932bce565ef70ed186655bc325e8543281050402c794a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gankLei-X/DeepION
  tools:
  - DeepION
  - PyTorch
  - Python
  - Resnet18
  - SimSiam
  - UMAP
  - kornia
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# simsiam-based-self-supervised-pipeline-integration

## Summary

Builds a SimSiam-style self-supervised contrastive pipeline for ion-image representation learning in mass spectrometry imaging, using paired augmentations of the same ion image and a stop-gradient loss to produce 512-dimensional representation vectors without manual annotation.

## When to use

Use when you need low-dimensional, annotation-free representations of ion images from an MSI matrix for downstream colocalized-ion or isotope-ion identification tasks.

## When NOT to use

- Data are raw, unpreprocessed MSI spectra rather than a preprocessed [X*Y, P] matrix and a [P,1] peak list — the pipeline expects preprocessed matrix/peak inputs.
- Labels or annotated pairs are already available and a supervised model is preferred — DeepION is designed for self-supervised representation without manual annotation.
- The task is general image representation outside MSI, where MSI-specific augmentation (Poisson noise, random/intensity-dependent missing value) is not appropriate.

## Inputs

- Preprocessed MSI matrix data with two-dimensional shape [X*Y, P] (X, Y pixel numbers of horizontal and vertical coordinates; P number of ions), e.g. DATASET/Pos_brain_data_matrix.txt
- Preprocessed MSI peak data with one-dimensional shape [P, 1], e.g. DATASET/Pos_brain_data_peak.csv
- Input MSI data shape (e.g. 198 422)
- Mode selection: 'COL' or 'ISO'
- Ion mode: 'positive' or 'negative'

## Outputs

- 20-dimensional dense representation vector O for each ion image, used for distance-based similarity measurement
- 512-dimensional representation vectors from the ResNet18-based encoders (intermediate)
- Output file of searched colocalized or isotope ions (e.g. Pos_COL_result, Pos_ISO_result)

## How to apply

Compose the pipeline as four modules: a data-augmentation module T that maps one original ion image to two augmented views (T_COL for colocalized ions, adding an intensity-dependent missing-value step as T_ISO for isotope ions), twin parameter-sharing ResNet18-based encoders that output 512-dimensional vectors, projection and prediction MLPs with a contrastive loss that maximizes similarity between the two augmentations under a stop-gradient operation to prevent collapse, and a dimensionality-reduction module (UMAP) that yields a 20-dimensional vector O by a trade-off between space–time complexity and information utilization. Feed the preprocessed MSI matrix of shape [X*Y, P] and the peak list of shape [P,1] into run.py with the selected COL or ISO mode, ion mode (positive/negative), and input shape (e.g. 198 422 for rat brain). Train with the Adam optimizer on the encoder and prediction module; validate by measuring colocalization/isotope discrimination against SIM-based, DR-based, and DL-based baselines.

## Related tools

- **DeepION** (End-to-end implementation of the SimSiam-based contrastive pipeline for ion-image representation in MSI) — https://github.com/gankLei-X/DeepION
- **SimSiam** (Base architecture on which DeepION is constructed (twin networks, contrastive loss with stop-gradient))
- **Resnet18** (Pretrained backbone network for the twin encoders)
- **PyTorch** (Python library in which the model is implemented)
- **UMAP** (Dimensionality reduction of the 512-d representation to the 20-d output vector)
- **kornia** (Dependency used by DeepION (acknowledged package))

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- Augmented views from the T module are well-formed tensors of the original ion-image shape and preserve the underlying spatial structure of the ion image.
- Simulated missing entries in augmented outputs have a frequency that increases with ion intensity (qualitative intensity-dependence check) in ISO mode.
- The reduced output dimension is 20, consistent with the stated trade-off between space–time complexity and information utilization.
- Benchmark: COL mode reaches ~86.73% classification accuracy on the colocalization task, ~25% above the second-best SimSiam model on rat brain tissue MSI.
- Stop-gradient contrastive training does not produce collapsed representations (two augmentations of the same image remain similar while distinct ions separate).

## Limitations

- Reduced dimension fixed to 20 — a trade-off that may discard information for some tasks.
- Requires preprocessed MSI matrix and peak list; raw spectra are not directly accepted.
- Only COL and ISO modes are implemented; other MSI-specific contrastive objectives are out of scope.
- DeepION's benefits depend on the specific augmentation strategy (Poisson noise, random/intensity-dependent missing value); misaligned augmentation may degrade representation quality.

## Evidence

- [methods] The proposed DeepION model is constructed based on the SimSiam model: "The proposed DeepION model is constructed based on the SimSiam model"
- [methods] It consists of twin networks that share identical parameters and adopt the pretrained Resnet18 as the backbone network: "It consists of twin networks that share identical parameters and adopt the pretrained Resnet18 as the backbone network"
- [methods] the reduced dimension is set to 20 by a trade-off between space–time complexity and information utilization: "the reduced dimension is set to 20 by a trade-off between space–time complexity and information utilization"
- [methods] The Adam optimizer is adopted to train the encoder and prediction module: "The Adam optimizer is adopted to train the encoder and prediction module"
- [readme] A contrastive loss is employed to maximize similarity with a stop-gradient operation to prevent collapsing during training.: [evidence span withheld — anchor status: external_doc]
- [readme] The original ion image is first imported into the data augmentation module "T" to generate two augmented images: "The original ion image is first imported into the data augmentation module "T"  to generate two augmented images"
- [readme] The preprocessed MSI matrix data with two-dimensional shape [X*Y,P]: "The preprocessed MSI matrix data with two-dimensional shape [X*Y,P], where X and Y represent the pixel numbers of horizontal and vertical coordinates of MSI data, and P represents the number of ions."
