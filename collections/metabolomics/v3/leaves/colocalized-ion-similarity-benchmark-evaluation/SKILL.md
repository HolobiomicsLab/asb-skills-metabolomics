---
name: colocalized-ion-similarity-benchmark-evaluation
description: Use when you must benchmark a new or existing ion-image representation
  for MSI colocalization on the rat brain positive-mode dataset and report leave-one-out
  classification accuracy versus the standard SIM/DR/DL baselines.
when_to_use_negative:
- If the goal is isotope ion discovery rather than regular colocalized ions from different
  molecules — use the ISO mode/intensity-dependent missing value workflow instead.
- If the MSI data are not preprocessed into a [X*Y, P] matrix and a [P, 1] peak list
  with a known pixel grid (e.g. 198 x 422), the benchmark cannot be run as specified.
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: DeepION
  role: SimSiam-based contrastive model with COL mode that generates the ion-image
    representation used for the colocalized-ion ranking benchmark
  repo: https://github.com/gankLei-X/DeepION
- name: SimSiam
  role: Base architecture on which the DeepION COL mode is constructed (encoder +
    projection + prediction MLPs with stop-gradient)
  repo: https://github.com/gankLei-X/DeepION
- name: Resnet18
  role: Pretrained backbone encoder (d=512) for both augmented views in the COL-mode
    twin network
- name: UMAP
  role: Dimensionality reduction module that maps the 512-dimensional representation
    to a 20-dimensional vector for similarity measurement
- name: PyTorch
  role: Implementation framework for the DeepION model training and inference
- name: Python
  role: Language in which the DeepION model and benchmark pipeline are implemented
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:1c0e2448b1c667ed714a8fd7c9ee1046659e984e2bfec8237750040b75175bda
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
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# colocalized-ion-similarity-benchmark-evaluation

## Summary

Evaluates a learned ion-image representation by ranking the top-5 colocalized ions per query ion on a rat brain MSI matrix and computing classification accuracy against SIM-based, DR-based, and DL-based baselines (Euclidean, cosine, PCC, SSIM, R2, PCA, t-SNE, UMAP, ResNet18, SimSiam).

## When to use

Use when you must benchmark a new or existing ion-image representation for MSI colocalization on the rat brain positive-mode dataset and report leave-one-out classification accuracy versus the standard SIM/DR/DL baselines.

## When NOT to use

- If the goal is isotope ion discovery rather than regular colocalized ions from different molecules — use the ISO mode/intensity-dependent missing value workflow instead.
- If the MSI data are not preprocessed into a [X*Y, P] matrix and a [P, 1] peak list with a known pixel grid (e.g. 198 x 422), the benchmark cannot be run as specified.

## Inputs

- Preprocessed rat brain MSI matrix in positive ion mode with two-dimensional shape [X*Y, P], e.g. DATASET/Pos_brain_data_matrix.txt (198 x 422 pixels)
- Preprocessed MSI peak list with one-dimensional shape [P, 1], e.g. DATASET/Pos_brain_data_peak.csv
- MSI input shape (198, 422), COL/ISO mode flag, positive/negative ion mode flag, number of colocalized ions to search per query (num=5)

## Outputs

- Top-5 colocalized-ion ranking per query ion based on the learned 20-dimensional representation
- Classification accuracy of DeepION COL mode versus Euclidean, cosine, PCC, SSIM, R2, PCA, t-SNE, UMAP, ResNet18, and SimSiam baselines (target ≈86.73%, ~25% above SimSiam)
- Output file Pos_COL_result with the COL-mode colocalization results

## How to apply

Load the preprocessed rat brain MSI matrix (198 x 422 pixels, shape [X*Y, P]) from DATASET/Pos_brain_data_matrix.txt and the peak list from DATASET/Pos_brain_data_peak.csv in positive ion mode. Apply the COL-mode data augmentation strategy (color jitter, filtering, Poisson noise, and random missing value) to generate augmented view pairs of each ion image for contrastive learning. Train the SimSiam-based DeepION model with a pretrained ResNet18 encoder (d=512) plus projection and prediction MLPs using the Adam optimizer (learning rate 0.0003, beta1 0.5, beta2 0.99, 100 iterations, stop-gradient to prevent collapse) in PyTorch. Reduce the 512-dimensional ion image representations to a 20-dimensional dense vector with UMAP and apply a min-max scaler to rescale features to [0, 1]. Rank the top 5 colocalized ions for each query ion by similarity on the learned representations and compute classification accuracy for DeepION COL mode against the Euclidean, cosine, PCC, SSIM, R2, PCA, t-SNE, UMAP, ResNet18, and SimSiam baselines. Write the COL-mode colocalization results to the output file Pos_COL_result. Validation: the COL-mode classification accuracy should reach approximately 86.73%, outperforming the second-best SimSiam baseline by about 25%.

## Related tools

- **DeepION** (SimSiam-based contrastive model with COL mode that generates the ion-image representation used for the colocalized-ion ranking benchmark) — https://github.com/gankLei-X/DeepION
- **SimSiam** (Base architecture on which the DeepION COL mode is constructed (encoder + projection + prediction MLPs with stop-gradient)) — https://github.com/gankLei-X/DeepION
- **Resnet18** (Pretrained backbone encoder (d=512) for both augmented views in the COL-mode twin network)
- **UMAP** (Dimensionality reduction module that maps the 512-dimensional representation to a 20-dimensional vector for similarity measurement)
- **PyTorch** (Implementation framework for the DeepION model training and inference)
- **Python** (Language in which the DeepION model and benchmark pipeline are implemented)

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- The COL-mode classification accuracy on the rat brain positive-mode benchmark should reach approximately 86.73% with leave-one-out validation.
- The DeepION COL-mode accuracy should outperform the second-best SimSiam baseline by about 25%.
- The output file Pos_COL_result should exist and contain the top-5 colocalized-ion rankings for the 198 x 422 pixel rat brain MSI matrix.
- Reduced representations should be a 20-dimensional dense vector, min–max scaled to [0, 1].

## Limitations

- The reduced dimension is set to 20 by a trade-off between space–time complexity and information utilization.
- The benchmark is specific to the preprocessed rat brain positive-mode dataset; other datasets, ion modes, or pixel grids require re-verification.
- Identifying colocalized ions is treated as vital to interpreting complex MSI data, but the approach requires self-supervised training on augmented view pairs; the article does not report limitations or weaknesses in the discussion section.

## Evidence

- [results] COL-mode accuracy ≈86.73% vs SimSiam baseline: "the COL mode of the DeepION model achieves 86.73%, which outperforms the second-best SimSiam model by about 25%"
- [abstract] Self-supervised contrastive learning for ion image representation: "contrastive learning is introduced to ensure that the model can generate the ion image representation in a self-supervised manner without manual annotation"
- [methods] MSI-specific data augmentation (color jitter, filtering, Poisson noise, random missing value): "including color jitter, filtering, Poisson noise, random missing value, and intensity-dependent missing value"
- [methods] Dimensionality reduction to 20-d dense vector via UMAP: "Since the UMAP algorithm has been demonstrated to perform better than other dimensionality reduction methods in many fields, we adopt it"
- [methods] Min–max scaler normalization to [0, 1]: "a min–max scaler is adopted to adjust the range of features to [0, 1]"
- [methods] SimSiam-based twin networks with pretrained ResNet18 backbone: "It consists of twin networks that share identical parameters and adopt the pretrained Resnet18 as the backbone network"
- [readme] UMAP with min–max scaler for the 20-dimensional representation: "Dimensionality Reduction module. This module is applied to further reduce the dimensionality of ion image representation to a 20-dimensional vector O for downstream tasks."
- [readme] COL-mode augmentation consists of color jitter, filtering, Poisson noise, random missing value: "T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode"
- [readme] SimSiam-based architecture with pretrained ResNet18 encoder and contrastive loss with stop-gradient: [evidence span withheld — anchor status: external_doc]
- [readme] COL-mode benchmark on the rat brain positive dataset uses Pos_brain_data_matrix.txt and Pos_brain_data_peak.csv: "python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file"
- [abstract] DeepION outperforms SIM-based, DR-based, and DL-based methods on the rat brain MSI benchmark: "Experimental results of rat brain tissue MSI show that DeepION outperforms other methods for both colocalized ion and isotope ion identification"
