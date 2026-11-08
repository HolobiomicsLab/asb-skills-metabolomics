---
name: self-supervised-contrastive-learning-training
description: Use when you need label-free representations of mass spectrometry imaging
  ion images (e.g. rat brain MSI matrices) to support downstream colocalized-ion or
  isotope-ion identification via distance-based similarity.
when_to_use_negative:
- When manually annotated training labels are already available and a supervised model
  is preferred over the self-supervised, annotation-free DeepION workflow.
- When the input is raw, un-preprocessed MSI data — DeepION expects a preprocessed
  matrix of shape [X*Y, P] and a preprocessed peak list, not raw instrument output.
- When the analysis target is neither colocalized-ion identification nor isotope-ion
  discovery (i.e. no COL/ISO mode applies).
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: DeepION
  role: The contrastive-learning model itself, implemented in two modes (COL and ISO)
    that generate ion image representations without manual annotation.
  repo: https://github.com/gankLei-X/DeepION
- name: SimSiam
  role: Architectural backbone on which DeepION is constructed, providing twin shared-parameter
    networks and the stop-gradient collapse-prevention mechanism.
- name: Resnet18
  role: Pretrained backbone encoder network used inside the twin encoders to extract
    512-dimensional representation vectors from augmented ion images.
- name: UMAP
  role: Dimensionality-reduction method that maps the 512-dimensional representations
    to the final 20-dimensional dense vector.
- name: PyTorch
  role: Python library in which the DeepION model is implemented and trained.
- name: kornia
  role: Acknowledged package used in the augmentation/training pipeline.
- name: boly_pytorch
  role: Acknowledged package used in the augmentation/training pipeline.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:e4719dede340dee7cb6214eba0e65974a30844dc3756078aed70440f45eee833
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
  - boly_pytorch
  - kornia
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9130434782608695
n_claims: 23
---

# self-supervised-contrastive-learning-training

## Summary

Train a SimSiam-based contrastive model (DeepION) that learns 512-dimensional ion image representations in a self-supervised manner by maximizing similarity between MSI-specific augmented views of each ion image, without manual annotation.

## When to use

Use when you need label-free representations of mass spectrometry imaging ion images (e.g. rat brain MSI matrices) to support downstream colocalized-ion or isotope-ion identification via distance-based similarity.

## When NOT to use

- When manually annotated training labels are already available and a supervised model is preferred over the self-supervised, annotation-free DeepION workflow.
- When the input is raw, un-preprocessed MSI data — DeepION expects a preprocessed matrix of shape [X*Y, P] and a preprocessed peak list, not raw instrument output.
- When the analysis target is neither colocalized-ion identification nor isotope-ion discovery (i.e. no COL/ISO mode applies).

## Inputs

- Preprocessed MSI matrix data with two-dimensional shape [X*Y, P] (e.g. Pos_brain_data_matrix.txt)
- Preprocessed MSI peak data with one-dimensional shape [P,1] (e.g. Pos_brain_data_peak.csv)
- MSI input shape (e.g. 198 422)
- Mode selection (COL or ISO)
- Ion mode selection (positive or negative)

## Outputs

- 512-dimensional ion image representation vectors from the ResNet18 encoder
- 20-dimensional dense vector O per ion image for downstream similarity measurement
- Min-max-scaled representation features in the range [0, 1]

## How to apply

Load the preprocessed MSI matrix ([X*Y, P], e.g. 198x422 pixels) and peak list (P,1) and generate augmented view pairs per ion image using a modality-specific augmentation module: T_COL applies color jitter, filtering, Poisson noise, and random missing value, while T_ISO adds intensity-dependent missing value to account for Poisson-distributed ion abundance and random missing patterns. Pass the two views through twin ResNet18-based encoders (d=512) that share parameters, then through projection and prediction MLPs; optimize with the Adam optimizer (learning rate 0.0003, beta1 0.5, beta2 0.99, 100 iterations) using a contrastive loss with a stop-gradient operation to prevent collapse, implemented in PyTorch. Reduce the resulting 512-dimensional representations to 20 dimensions with UMAP and rescale with a min-max scaler to [0, 1] before downstream similarity measurement.

## Related tools

- **DeepION** (The contrastive-learning model itself, implemented in two modes (COL and ISO) that generate ion image representations without manual annotation.) — https://github.com/gankLei-X/DeepION
- **SimSiam** (Architectural backbone on which DeepION is constructed, providing twin shared-parameter networks and the stop-gradient collapse-prevention mechanism.)
- **Resnet18** (Pretrained backbone encoder network used inside the twin encoders to extract 512-dimensional representation vectors from augmented ion images.)
- **UMAP** (Dimensionality-reduction method that maps the 512-dimensional representations to the final 20-dimensional dense vector.)
- **PyTorch** (Python library in which the DeepION model is implemented and trained.)
- **kornia** (Acknowledged package used in the augmentation/training pipeline.)
- **boly_pytorch** (Acknowledged package used in the augmentation/training pipeline.)

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- The COL-mode classification accuracy on the rat brain benchmark reaches approximately 86.73%, outperforming the second-best SimSiam model by about 25%.
- The encoder outputs two 512-dimensional representation vectors, one per augmented view, confirming the ResNet18-based twin architecture is wired correctly.
- After dimensionality reduction, each ion image yields a 20-dimensional vector O, matching the stated trade-off between space-time complexity and information utilization.
- Min-max scaler output features fall within [0, 1], confirming normalization was applied as specified.
- Training loss does not collapse to a degenerate solution, confirming the stop-gradient operation is functioning during optimization.

## Limitations

- The reduced dimension is set to 20 by a trade-off between space–time complexity and information utilization; other dimensionalities are not validated in the article.
- The augmentation strategies are modality-specific: T_COL omits intensity-dependent missing value, which is only present in T_ISO; choosing the wrong mode may misrepresent the missing-value characteristics.
- The DeepION model is trained on a workstation equipped with an Nvidia GTX 2080Ti GPU; resource-constrained environments may need adjustments.
- Figure S1 (referenced in the discussion) is not included in the extracted text, so fine differences between colocalized and isotope ions must be checked in the SI.

## Evidence

- [abstract] contrastive learning is introduced to ensure that the model can generate the ion image representation in a self-supervised manner without manual annotation: "contrastive learning is introduced to ensure that the model can generate the ion image representation in a self-supervised manner without manual annotation"
- [methods] a modality-specific data augmentation strategy is designed by incorporating the characteristics of MSI data: "a modality-specific data augmentation strategy is designed by incorporating the characteristics of MSI data"
- [methods] It consists of twin networks that share identical parameters and adopt the pretrained Resnet18 as the backbone network: "It consists of twin networks that share identical parameters and adopt the pretrained Resnet18 as the backbone network"
- [methods] The Adam optimizer is adopted to train the encoder and prediction module: "The Adam optimizer is adopted to train the encoder and prediction module"
- [readme] the T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode: "the T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode"
- [readme] A contrastive loss is employed to maximize similarity with a stop-gradient operation to prevent collapsing during training: [evidence span withheld — anchor status: external_doc]
- [methods] Since the UMAP algorithm has been demonstrated to perform better than other dimensionality reduction methods in many fields, we adopt it: "Since the UMAP algorithm has been demonstrated to perform better than other dimensionality reduction methods in many fields, we adopt it"
- [methods] the reduced dimension is set to 20 by a trade-off between space–time complexity and information utilization: "the reduced dimension is set to 20 by a trade-off between space–time complexity and information utilization"
