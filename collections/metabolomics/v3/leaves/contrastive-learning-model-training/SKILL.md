---
name: contrastive-learning-model-training
description: Use when you need annotation-free dense vector representations of ion
  images from preprocessed MSI pixel-by-ion matrices for downstream colocalized-ion
  or isotope-ion retrieval tasks.
when_to_use_negative:
- When manually labeled training data are available and a supervised classifier is
  preferable — DeepION is designed for self-supervised representation learning without
  manual annotation.
- When the goal is isotope ion identification rather than colocalized ion retrieval
  and the input characteristics demand the ISO-mode augmentation (intensity-dependent
  missing value) — do not reuse COL-mode augmentations for that task.
- When the input is already a reduced feature/embedding table rather than a raw preprocessed
  MSI pixel-by-ion matrix — the contrastive training pipeline expects unembedded ion
  images.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: DeepION
  role: The SimSiam-based contrastive representation model being trained, in COL or
    ISO mode
  repo: https://github.com/gankLei-X/DeepION
- name: SimSiam
  role: Base contrastive architecture on which DeepION is constructed (stop-gradient,
    projection/prediction MLPs)
- name: Resnet18
  role: Pretrained backbone encoder extracting 512-d features from augmented ion images
- name: UMAP
  role: Dimensionality reduction of encoder outputs to a 20-dimensional vector for
    downstream similarity measurement
- name: PyTorch
  role: Implementation library for the DeepION model and training loop
- name: Python
  role: Runtime language for running run.py and the training pipeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:98db536c4986495e2c621543b398e53bcdb590c647ebf6e802dcc62c43d78653
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gankLei-X/DeepION
  tools:
  - DeepION
  - Nvidia GTX 2080Ti
  - PyTorch
  - Python
  - Resnet18
  - SimSiam
  - UMAP
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# contrastive-learning-model-training

## Summary

Train a SimSiam-style self-supervised contrastive model (DeepION) that learns low-dimensional representations of ion images from mass spectrometry imaging data, using MSI-specific data augmentation and a stop-gradient mechanism to avoid collapse without manual annotation.

## When to use

Use when you need annotation-free dense vector representations of ion images from preprocessed MSI pixel-by-ion matrices for downstream colocalized-ion or isotope-ion retrieval tasks.

## When NOT to use

- When manually labeled training data are available and a supervised classifier is preferable — DeepION is designed for self-supervised representation learning without manual annotation.
- When the goal is isotope ion identification rather than colocalized ion retrieval and the input characteristics demand the ISO-mode augmentation (intensity-dependent missing value) — do not reuse COL-mode augmentations for that task.
- When the input is already a reduced feature/embedding table rather than a raw preprocessed MSI pixel-by-ion matrix — the contrastive training pipeline expects unembedded ion images.

## Inputs

- Preprocessed MSI matrix data, 2D shape [X*Y, P] (e.g. DATASET/Pos_brain_data_matrix.txt, 198x422 pixels)
- Preprocessed MSI peak list, 1D shape [P,1] (e.g. DATASET/Pos_brain_data_peak.csv)
- MSI data input shape (horizontal and vertical pixel counts)
- Mode selection string: 'COL' or 'ISO'
- Ion mode string: 'positive' or 'negative'
- Number of colocalized ions to search per ion (e.g. 5)

## Outputs

- 20-dimensional UMAP-reduced ion image representation vectors
- Min–max scaled (to [0,1]) feature matrix for distance-based similarity measurement
- Top-k colocalized ion search results per ion (e.g. Pos_COL_result output file)
- Trained SimSiam-based Resnet18 encoder checkpoint

## How to apply

Load the preprocessed MSI matrix (shape [X*Y, P], e.g. DATASET/Pos_brain_data_matrix.txt, 198x422 pixels) and the peak list (Pos_brain_data_peak.csv). Train DeepION in the desired mode (COL for colocalized ions from different molecules, ISO for isotope ions from the same molecule): twin networks share parameters with a pretrained Resnet18 encoder producing 512-d vectors, a three-layer FC projection MLP, and a two-layer FC prediction MLP; apply MSI-specific augmentations (T_COL: color jitter, filtering, Poisson noise, random missing value; T_ISO additionally uses intensity-dependent missing value) to generate two augmented views per ion image, and use a contrastive loss with a stop-gradient operation to prevent representation collapse. Train with the Adam optimizer (learning rate 0.0003, beta1=0.5, beta2=0.99), N(0,0.02) weight initialization, batch size limited by GPU memory, for 100 iterations on e.g. an Nvidia GTX 2080Ti. Reduce encoder outputs to a 20-dimensional UMAP embedding, min–max scale features to [0,1], and perform distance-based similarity retrieval of top colocalized ions (top-5 in COL mode). The MSI-specific augmentations encode MSI priors (Poisson distribution of ion abundance, random missing patterns) and are the key differentiator: ablations show they account for roughly a 25 percentage point accuracy gain over vanilla SimSiam (86.73% vs ~62% leave-one-out on 98 annotated ion images).

## Related tools

- **DeepION** (The SimSiam-based contrastive representation model being trained, in COL or ISO mode) — https://github.com/gankLei-X/DeepION
- **SimSiam** (Base contrastive architecture on which DeepION is constructed (stop-gradient, projection/prediction MLPs))
- **Resnet18** (Pretrained backbone encoder extracting 512-d features from augmented ion images)
- **UMAP** (Dimensionality reduction of encoder outputs to a 20-dimensional vector for downstream similarity measurement)
- **PyTorch** (Implementation library for the DeepION model and training loop)
- **Python** (Runtime language for running run.py and the training pipeline)

## Examples

```
python run.py --input_Matrix DATASET/Pos_brain_data_matrix.txt --input_PeakList DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- In COL mode, the leave-one-out linear classifier on 98 manually annotated ion images (17 categories) should reach ~86.73% accuracy, about 25 percentage points above a vanilla SimSiam baseline trained with only natural-image augmentations (color jitter, filtering).
- The encoder must not produce collapse results — verify that the stop-gradient prediction MLP keeps representations non-degenerate (distinct vectors per ion image).
- Encoder output dimensionality must be 512 before reduction and exactly 20 after UMAP reduction, with features min–max scaled to [0,1].
- Top-5 distance-based retrieval of colocalized ions from the embedding space should outperform SIM-based (PCC, R2) and other DR/DL baselines on rat brain MSI.
- Training logs should confirm Adam optimizer with lr=0.0003 (beta1=0.5, beta2=0.99), N(0,0.02) weight init, and completion of the 100-iteration schedule within GPU memory limits.

## Limitations

- Batch size is limited by GPU memory (Nvidia GTX 2080Ti in the original work), constraining scaling to larger MSI datasets.
- Performance depends critically on the MSI-specific augmentation scheme; omitting Poisson-noise and random-missing-value augmentations (as in vanilla SimSiam) costs roughly 25 percentage points of accuracy.
- The reduced dimension of 20 is a stated trade-off between space–time complexity and information utilization; other dimensionalities are not validated.
- The distinction between COL and ISO modes is subtle (Figure S1 indicates the fine difference); applying the wrong mode's augmentation risks conflating isotope ions with colocalized ions from different molecules.

## Evidence

- [abstract] Self-supervised contrastive training: "contrastive learning is introduced to ensure that the model can generate the ion image representation in a self-supervised manner without manual annotation"
- [abstract] MSI-specific augmentation: "a unique data augmentation strategy is designed by considering the characteristics of MSI data, such as the Poisson distribution of ion abundance and a random pattern of missing values"
- [readme] Architecture and stop-gradient: "Two augmented images are propagated through a pair of ResNet18-based encoders that shared parameters, then output two 512-dimensional representation vectors"
- [readme] COL augmentation set: "the T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode, while T_ISO introduces an additional process of intensity-dependent missing value"
- [results] Benchmark accuracy: "the COL mode of the DeepION model achieves 86.73%, which outperforms the second-best SimSiam model by about 25%"
- [methods] Dimensionality reduction choice: "the reduced dimension is set to 20 by a trade-off between space–time complexity and information utilization"
