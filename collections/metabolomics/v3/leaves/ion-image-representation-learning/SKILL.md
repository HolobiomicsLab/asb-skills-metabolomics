---
name: ion-image-representation-learning
description: Use when MSI pixel-by-ion matrices (e.g. [X*Y,P] shape, 198x422 rat brain
  images) must be converted into dense ion-image representations for colocalized-ion
  or isotope-ion search without manual annotation.
when_to_use_negative:
- When labeled training data are available and a supervised representation could be
  learned — DeepION is designed to generate representations in a self-supervised manner
  without manual annotation.
- When the goal is molecular identification from raw spectra rather than ion-image
  spatial representations; DeepION targets ion images, not peak-level annotation.
- When pixel-by-ion matrices have not been preprocessed — DeepION expects two-dimensional
  [X*Y,P] MSI matrices and a [P,1] peak list, not raw instrument acquisitions.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: DeepION
  role: Main model implementing the SimSiam-based self-supervised encoder, projection,
    prediction, and dimensionality-reduction pipeline for ion images.
  repo: https://github.com/BioNet-XMU/DeepION
- name: SimSiam
  role: Contrastive learning framework whose stop-gradient operation underpins DeepION's
    loss to prevent collapsing solutions.
- name: Resnet18
  role: Pretrained backbone for the twin encoder networks that output 512-dimensional
    ion image representations.
- name: UMAP
  role: Reduces the 512-dimensional representations to 20-dimensional dense vectors
    for downstream similarity measurement.
- name: PyTorch
  role: Python deep-learning library used to implement and train DeepION.
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:d33da9940a08a8e91f8ca94a25bc959c279b45b39cad116ea33055c007bccad6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioNet-XMU/DeepION
  tools:
  - DeepION
  - METASPACE
  - Nvidia GTX 2080Ti
  - PCC
  - PyTorch
  - Python
  - R2
  - Resnet18
  - SimSiam
  - UMAP
  - rMSI
  - torchvision
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9259259259259259
n_claims: 27
---

# ion-image-representation-learning

## Summary

Train DeepION, a SimSiam-based self-supervised contrastive model, to compress mass spectrometry imaging (MSI) ion images into 512-dimensional representation vectors that are further reduced to 20-dimensional dense vectors for distance-based ion colocalization and isotope identification.

## When to use

Use when MSI pixel-by-ion matrices (e.g. [X*Y,P] shape, 198x422 rat brain images) must be converted into dense ion-image representations for colocalized-ion or isotope-ion search without manual annotation.

## When NOT to use

- When labeled training data are available and a supervised representation could be learned — DeepION is designed to generate representations in a self-supervised manner without manual annotation.
- When the goal is molecular identification from raw spectra rather than ion-image spatial representations; DeepION targets ion images, not peak-level annotation.
- When pixel-by-ion matrices have not been preprocessed — DeepION expects two-dimensional [X*Y,P] MSI matrices and a [P,1] peak list, not raw instrument acquisitions.

## Inputs

- Preprocessed MSI matrix data with two-dimensional shape [X*Y,P]
- Preprocessed MSI peak list data with one-dimensional shape [P,1]
- MSI data shape (e.g. 198 422 for the rat brain section)
- Mode selection: COL or ISO; ion mode: positive or negative

## Outputs

- 512-dimensional ion image representation vectors from the encoder
- 20-dimensional dense vector O per ion after UMAP dimensionality reduction and min-max scaling
- Colocalized-ion search results (e.g. Pos_COL_result) or isotope pair discovery results (e.g. Pos_ISO_result)

## How to apply

Run the data augmentation module T on each ion image to generate two augmented views: in COL mode apply color jitter, filtering, Poisson noise, and random missing value; in ISO mode add an intensity-dependent missing value suited to isotope ions of the same molecule. Feed the two views through parameter-sharing ResNet18-based encoders producing 512-dimensional vectors r1 and r2, then through a three-layer MLP projection and a two-layer MLP prediction MLP that, together with the SimSiam contrastive loss with stop-gradient operation, prevents collapse. Initialize all parameters from N(0, 0.02) and train with Adam (learning rate 0.0003, beta1 0.5, beta2 0.99, 100 iterations) in PyTorch on an Nvidia GTX 2080Ti. Reduce the learned representations with UMAP to a 20-dimensional vector and rescale to [0,1] with a min-max scaler before distance-based similarity measurement for downstream identification.

## Related tools

- **DeepION** (Main model implementing the SimSiam-based self-supervised encoder, projection, prediction, and dimensionality-reduction pipeline for ion images.) — https://github.com/BioNet-XMU/DeepION
- **SimSiam** (Contrastive learning framework whose stop-gradient operation underpins DeepION's loss to prevent collapsing solutions.)
- **Resnet18** (Pretrained backbone for the twin encoder networks that output 512-dimensional ion image representations.)
- **UMAP** (Reduces the 512-dimensional representations to 20-dimensional dense vectors for downstream similarity measurement.)
- **PyTorch** (Python deep-learning library used to implement and train DeepION.)

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- Encoder outputs must be 512-dimensional (d = 512) representation vectors for each augmented view.
- Dimensionality-reduction module must produce a 20-dimensional vector per ion image before min-max scaling to [0,1].
- Training must use the SimSiam contrastive loss with the stop-gradient operation to avoid collapse results.
- On the rat brain COL benchmark the pipeline should support reaching 86.73% colocalization accuracy, and in ISO mode 442 pairs with 92.76% accuracy in positive mode (249 pairs, 75.90% in negative mode).
- The run.py invocation with --input_Matrix .../DATASET/Pos_brain_data_matrix.txt and --input_shape 198 422 must execute end-to-end producing Pos_COL_result or Pos_ISO_result outputs.

## Limitations

- Requires preprocessed MSI matrix and peak list inputs with the specified shapes; raw instrument data must be converted beforehand.
- The reduced dimension is set to 20 by a trade-off between space–time complexity and information utilization, so finer information may be lost.
- Model parameters are initialized from N(0, 0.02) and trained for 100 iterations, which may need adaptation for other MSI datasets.
- Compute is modest but GPU-dependent; the reported setup used a workstation equipped with a GPU Nvidia GTX 2080Ti graphics card.

## Evidence

- [methods] SimSiam twin encoder with ResNet18 backbone: "It consists of twin networks that share identical parameters and adopt the pretrained Resnet18 as the backbone network"
- [readme] COL/ISO augmentation composition: "the T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode, while T_ISO introduces an additional process of intensity-dependent missing value"
- [readme] Stop-gradient contrastive loss prevents collapse: [evidence span withheld — anchor status: external_doc]
- [readme] UMAP to 20-dimensional vector: "This module is applied to further reduce the dimensionality of ion image representation to a 20-dimensional vector O for downstream tasks"
- [results] COL-mode benchmark accuracy: "the COL mode of the DeepION model achieves 86.73%, which outperforms the second-best SimSiam model by about 25%"
- [readme] Concrete run.py invocation for COL mode: "python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file"
