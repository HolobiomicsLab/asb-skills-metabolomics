---
name: self-supervised-ion-image-representation-learning
description: Use when raw preprocessed MSI matrices (shape [X*Y, P], e.
when_to_use_negative:
- Input is raw, unpreprocessed mass-spectrometry imaging data rather than a preprocessed
  [X*Y, P] intensity matrix with a matching peak list — the model expects pixel-summed
  ion intensities, not vendor spectra or imaging files.
- 'The goal is general image classification or supervised annotation rather than self-supervised
  ion-image representation: DeepION requires no manual annotation, and its contrastive
  objective is not designed to learn labeled class boundaries.'
- The task only needs isotope-partner ranking on data where PCC or R2 already succeed
  and no missing-value/low-SNR challenge exists — in that case a simpler similarity
  computation may suffice, though DeepION was shown to outperform PCC and R2 exactly
  where those fail.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3474
tools:
- name: DeepION
  role: Main framework implementing COL/ISO modes for self-supervised ion-image representation
  repo: https://github.com/BioNet-XMU/DeepION
- name: SimSiam
  role: 'Base architecture: twin networks with stop-gradient contrastive loss to avoid
    collapse'
  repo: https://github.com/gankLei-X/DeepION
- name: Resnet18
  role: Pretrained backbone for the twin encoders producing 512-d representations
  repo: https://github.com/gankLei-X/DeepION
- name: UMAP
  role: Dimensionality-reduction module mapping encoder outputs to 20-d vectors
- name: PyTorch
  role: Implementation library for the model
- name: kornia
  role: Supports data-augmentation operations (color jitter, filtering)
- name: boly_pytorch
  role: Acknowledged supporting package used by DeepION
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:d9e87df810f8935c28340118a486ac792d741d993def76f5e22743ed2da40251
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioNet-XMU/DeepION
  tools:
  - DeepION
  - PyTorch
  - Python
  - Resnet18
  - SimSiam
  - UMAP
  - boly_pytorch
  - kornia
  - rMSI
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 18
---

# self-supervised-ion-image-representation-learning

## Summary

Trains DeepION, a SimSiam-based contrastive model with a pretrained Resnet18 backbone, to produce 20-dimensional self-supervised representations of mass spectrometry imaging (MSI) ion images that preserve spectral and spatial structure for colocalized-ion and isotope identification without manual annotation.

## When to use

Use when raw preprocessed MSI matrices (shape [X*Y, P], e.g. 198 x 422 rat brain pixels with a peak list of P ions) must be converted into dense ion-image embeddings to rank colocalized ions or discover isotope partners, especially where missing values and low SNR defeat similarity methods like PCC or R2.

## When NOT to use

- Input is raw, unpreprocessed mass-spectrometry imaging data rather than a preprocessed [X*Y, P] intensity matrix with a matching peak list — the model expects pixel-summed ion intensities, not vendor spectra or imaging files.
- The goal is general image classification or supervised annotation rather than self-supervised ion-image representation: DeepION requires no manual annotation, and its contrastive objective is not designed to learn labeled class boundaries.
- The task only needs isotope-partner ranking on data where PCC or R2 already succeed and no missing-value/low-SNR challenge exists — in that case a simpler similarity computation may suffice, though DeepION was shown to outperform PCC and R2 exactly where those fail.

## Inputs

- Preprocessed MSI matrix with two-dimensional shape [X*Y, P] (e.g. DATASET/Pos_brain_data_matrix.txt, 198 x 422 pixels)
- Preprocessed MSI peak data with one-dimensional shape [P, 1] (e.g. DATASET/Pos_brain_data_peak.csv)
- MSI data shape (X, Y pixel counts)
- Mode selection: 'COL' (colocalized ions) or 'ISO' (isotope ions)
- Ion mode: 'positive' or 'negative'
- Number of searched colocalized ions per ion (COL mode only, e.g. --num 5)

## Outputs

- 20-dimensional ion image representation vectors (min–max scaled to [0, 1]) per ion
- Intermediate 512-dimensional encoder representations before UMAP reduction
- Ranked neighbor lists of colocalized or isotope ions per query ion (e.g. Pos_COL_result / Pos_ISO_result output file)

## How to apply

Load the preprocessed MSI intensity matrix [X*Y, P] and matching peak list (e.g. DATASET/Pos_brain_data_matrix.txt with Pos_brain_data_peak.csv for 198 x 422 pixels) and select mode COL for colocalized ions from different molecules or ISO for isotope ions from the same molecule. Run DeepION in Python/PyTorch: the data-augmentation module generates two augmented views per ion image—COL uses color jitter, filtering, Poisson noise and random missing value, while ISO adds intensity-dependent missing value reflecting the MSI missing-value pattern—and the augmented pairs pass through twin Resnet18 encoders with shared parameters, producing 512-dimensional representation vectors. A projection MLP, a prediction MLP and a contrastive loss with stop-gradient (SimSiam architecture) maximize similarity between the two augmentations to avoid collapse during self-supervised training; train with the Adam optimizer. Feed encoder outputs to the UMAP-based dimensionality-reduction module to obtain 20-dimensional vectors (chosen by a trade-off between space–time complexity and information utilization), then min–max scale the features to [0, 1]. Downstream, use distance-based similarity over these embeddings to rank candidate colocalized or isotope ions for each query m/z.

## Related tools

- **DeepION** (Main framework implementing COL/ISO modes for self-supervised ion-image representation) — https://github.com/BioNet-XMU/DeepION
- **SimSiam** (Base architecture: twin networks with stop-gradient contrastive loss to avoid collapse) — https://github.com/gankLei-X/DeepION
- **Resnet18** (Pretrained backbone for the twin encoders producing 512-d representations) — https://github.com/gankLei-X/DeepION
- **UMAP** (Dimensionality-reduction module mapping encoder outputs to 20-d vectors)
- **PyTorch** (Implementation library for the model)
- **kornia** (Supports data-augmentation operations (color jitter, filtering))
- **boly_pytorch** (Acknowledged supporting package used by DeepION)

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode ISO --ion_mode positive --output_file Pos_ISO_result
```

## Evaluation signals

- COL mode reaches ~86.73% classification accuracy on the benchmark colocalization task, about 25% higher than the second-best SimSiam model — a reproduction should approach this accuracy.
- The reduced representation dimension is fixed at 20; verify UMAP output vectors have exactly 20 features and are min–max scaled to [0, 1].
- For ISO mode on rat brain negative-mode data, DeepION must retrieve the reported isotope partners (e.g. m/z 1547.8702 for query 1544.847) among top-ranked neighbors where PCC < 0.5 and R2 < 0.
- Matrix input shape must match the declared --input_shape (e.g. 198 422) and peak-list length must equal P of the matrix.
- Distance-based similarity ranking over the 20-d embeddings should outperform SIM-based baselines (PCC, R2) on rat brain tissue MSI for both colocalized and isotope identification.

## Limitations

- Requires preprocessed two-dimensional [X*Y, P] matrix input; raw MSI vendor files are not directly consumed.
- Performance depends on augmentation choices matched to MSI characteristics (Poisson-distributed ion abundance and random missing patterns); misapplied augmentation (e.g. using COL augmentation for isotope discovery) can degrade representation quality.
- Trained on a workstation GPU (Nvidia GTX 2080Ti); PyTorch 1.8.2 and umap 0.5.1 pins mean environment drift may affect reproducibility.
- The article flags that the discussion section provides no quantitative comparison metrics, training hyperparameters, or dataset accessions beyond those in methods/results.

## Evidence

- [abstract] contrastive learning is introduced to ensure that the model can generate the ion image representation in a self-supervised manner without manual annotation: "contrastive learning is introduced to ensure that the model can generate the ion image representation in a self-supervised manner without manual annotation"
- [methods] The proposed DeepION model is constructed based on the SimSiam model: "The proposed DeepION model is constructed based on the SimSiam model"
- [methods] It consists of twin networks that share identical parameters and adopt the pretrained Resnet18 as the backbone network: "It consists of twin networks that share identical parameters and adopt the pretrained Resnet18 as the backbone network"
- [readme] The preprocessed MSI matrix data with two-dimensional shape [X*Y,P], where X and Y represent the pixel numbers of horizontal and vertical coordinates of MSI data, and P represents the number of ions.: "The preprocessed MSI matrix data with two-dimensional shape [X*Y,P], where X and Y represent the pixel numbers of horizontal and vertical coordinates of MSI data, and P represents the number of ions."
- [readme] T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode, while T_ISO introduces an additional process of intensity-dependent missing value in ISO mode: "T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode, while T_ISO introduces an additional process of intensity-dependent missing value in ISO mode"
- [readme] This module is applied to further reduce the dimensionality of ion image representation to a 20-dimensional vector O for downstream tasks.: "This module is applied to further reduce the dimensionality of ion image representation to a 20-dimensional vector O for downstream tasks."
- [results] the COL mode of the DeepION model achieves 86.73%, which outperforms the second-best SimSiam model by about 25%: "the COL mode of the DeepION model achieves 86.73%, which outperforms the second-best SimSiam model by about 25%"
