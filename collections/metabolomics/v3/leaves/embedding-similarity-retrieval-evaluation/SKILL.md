---
name: embedding-similarity-retrieval-evaluation
description: Use when you have learned dense representations of ion images from an
  MSI dataset and need to quantify embedding quality via colocalized-ion retrieval
  or leave-one-out linear classification.
when_to_use_negative:
- Input is raw, unrepresented ion intensity data rather than learned embedding vectors
  — train an encoder (e.g., DeepION) first
- The goal is isotope ion discovery from the same molecule (DeepION ISO mode) rather
  than colocalization across different molecules (COL mode)
- You lack ground-truth colocalization annotations, making retrieval accuracy and
  leave-one-out classification unverifiable
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3382
tools:
- name: DeepION
  role: Source of the learned ion-image embeddings (COL mode, SimSiam-based architecture
    with pretrained ResNet18 encoder) that are evaluated by retrieval
  repo: https://github.com/gankLei-X/DeepION
- name: SimSiam
  role: Baseline self-supervised model whose embedding space is compared against DeepION's
    via the same retrieval protocol
- name: UMAP
  role: Dimensionality-reduction module used to project encoder outputs to the 20-dimensional
    dense vector used for similarity measurement
- name: PyTorch
  role: Implementation library for the DeepION model and training pipeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:4fabd8a3f6a5a0b1ca37aef6c7c6091f2dc9c715ab7d7e9f2bd6143d607af5fb
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
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.92
n_claims: 25
---

# embedding-similarity-retrieval-evaluation

## Summary

Evaluates the quality of learned ion-image embeddings from MSI data by reducing encoder outputs to 20-dimensional UMAP vectors, min–max scaling features to [0,1], and performing distance-based retrieval of the top-5 colocalized ions per ion — the protocol DeepION uses to benchmark embedding spaces against SimSiam.

## When to use

Use when you have learned dense representations of ion images from an MSI dataset and need to quantify embedding quality via colocalized-ion retrieval or leave-one-out linear classification.

## When NOT to use

- Input is raw, unrepresented ion intensity data rather than learned embedding vectors — train an encoder (e.g., DeepION) first
- The goal is isotope ion discovery from the same molecule (DeepION ISO mode) rather than colocalization across different molecules (COL mode)
- You lack ground-truth colocalization annotations, making retrieval accuracy and leave-one-out classification unverifiable

## Inputs

- Learned encoder output vectors for ion images (e.g., 512-dim ResNet18 representations)
- Preprocessed MSI matrix with shape [X*Y, P] (e.g., DATASET/Pos_brain_data_matrix.txt, 198x422 pixels)
- Preprocessed MSI peak list with shape [P,1] (e.g., DATASET/Pos_brain_data_peak.csv)
- Manually annotated colocalization labels for benchmark evaluation (98 ion images, 17 categories)

## Outputs

- 20-dimensional UMAP embedding of ion image representations
- Min–max scaled feature matrix in range [0, 1]
- Top-5 colocalized ion retrieval lists per ion (e.g., Pos_COL_result output file)
- Ablation comparison table (e.g., augmentation_ablation_comparison.csv) quantifying retrieval accuracy across models

## How to apply

First reduce each model's encoder outputs (e.g., the 512-dim ResNet18 representations from DeepION's COL mode) to a 20-dimensional UMAP embedding — the paper sets the reduced dimension to 20 as a trade-off between space–time complexity and information utilization. Next apply a min–max scaler to adjust each feature's range to [0, 1] so that distance computations are not dominated by large-intensity features. Then perform distance-based similarity measurement, retrieving the top-N (N=5 in the COL-mode run configuration) most similar ions for each ion and comparing the retrieved lists against known colocalized ion pairs. Validate embedding quality by reproducing DeepION COL-mode's reported 86.73% leave-one-out classification accuracy on the 98-image, 17-category benchmark, which exceeds the SimSiam baseline by about 25 percentage points — a gap attributable to MSI-specific augmentation (Poisson noise, random missing value) versus standard natural-image augmentation.

## Related tools

- **DeepION** (Source of the learned ion-image embeddings (COL mode, SimSiam-based architecture with pretrained ResNet18 encoder) that are evaluated by retrieval) — https://github.com/gankLei-X/DeepION
- **SimSiam** (Baseline self-supervised model whose embedding space is compared against DeepION's via the same retrieval protocol)
- **UMAP** (Dimensionality-reduction module used to project encoder outputs to the 20-dimensional dense vector used for similarity measurement)
- **PyTorch** (Implementation library for the DeepION model and training pipeline)

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- DeepION COL-mode retrieval/classification accuracy reproduces or approximates the reported 86.73% leave-one-out accuracy on the 98-image benchmark
- The gap over the SimSiam baseline is approximately 25 percentage points, confirming MSI-specific augmentation (Poisson noise, random missing value) as the source of the advantage
- All output embedding features are min–max scaled to exactly the range [0, 1]
- Embedding dimensionality after UMAP reduction is 20, matching the paper's stated space–time complexity trade-off
- Retrieval returns exactly the top-5 colocalized ions per ion, matching the COL-mode --num 5 configuration

## Limitations

- Benchmark evaluation relies on manually annotated data: only 98 manually annotated ion images across 17 categories were used for the leave-one-out classifier
- The 20-dimensional reduction is a stated trade-off, so some information utilization is sacrificed relative to higher-dimensional representations
- Training details such as batch size are limited by GPU memory (Nvidia GTX 2080Ti), which may constrain reproducibility on smaller hardware
- The article's discussion section provides no quantitative metrics or baseline identifiers for the comparison against previous methods, limiting cross-paper validation

## Evidence

- [abstract] Distance-based similarity measurement for identification of colocalized ions: "which further facilitates the distance-based similarity measurement for the identification of colocalized ions"
- [methods] UMAP dimensionality reduction: "Since the UMAP algorithm has been demonstrated to perform better than other dimensionality reduction methods in many fields, we adopt it"
- [methods] Reduced dimension choice: "the reduced dimension is set to 20 by a trade-off between space–time complexity and information utilization"
- [results] Benchmark accuracy result: "the COL mode of the DeepION model achieves 86.73%, which outperforms the second-best SimSiam model by about 25%"
- [readme] Retrieval count configuration: "(6) The number of searched co-locolized ions for each ion."
