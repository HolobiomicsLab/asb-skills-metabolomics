---
name: umap-dimensionality-reduction
description: Use when learned ion-image embeddings (e.g., 512-d vectors from a contrastive
  encoder) must be compressed to a low-dimensional dense vector before ranking top
  colocalized ions by distance in an MSI analysis.
when_to_use_negative:
- The representation is already a low-dimensional feature table (reducing again risks
  discarding the information the 20-d trade-off was calibrated for).
- The task is isotope-ion identification with DeepION's ISO mode where the goal is
  intensity-dependent missing-value augmentation rather than COL-mode colocalization
  retrieval.
- You intend to train or evaluate the encoder itself — UMAP is a post-hoc dimensionality-reduction
  step, not a training objective.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: UMAP
  role: Dimensionality reduction of the 512-d encoder output to a 20-d dense vector
    for downstream similarity measurement (umap 0.5.1 per README requirements).
- name: DeepION
  role: Upstream SimSiam-based contrastive model whose COL-mode encoder produces the
    512-d ion-image representations that UMAP reduces.
  repo: https://github.com/gankLei-X/DeepION
- name: scikit-learn
  role: Provides the min–max scaler that rescales the 20-d UMAP output features to
    [0, 1] before distance computation.
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:2df310ba4ec8e2dabea9106b587b6cf44520b4ed9c874ed82c50f3bc699de050
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gankLei-X/DeepION
  tools:
  - DeepION
  - Nvidia GTX 2080Ti
  - Python
  - Resnet18
  - UMAP
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# umap-dimensionality-reduction

## Summary

Reduce high-dimensional ion-image representations (e.g., 512-d ResNet18 encoder outputs from DeepION) to a 20-dimensional dense vector with UMAP, then min–max scale to [0,1] so distance-based colocalized-ion similarity retrieval can be performed.

## When to use

Use when learned ion-image embeddings (e.g., 512-d vectors from a contrastive encoder) must be compressed to a low-dimensional dense vector before ranking top colocalized ions by distance in an MSI analysis.

## When NOT to use

- The representation is already a low-dimensional feature table (reducing again risks discarding the information the 20-d trade-off was calibrated for).
- The task is isotope-ion identification with DeepION's ISO mode where the goal is intensity-dependent missing-value augmentation rather than COL-mode colocalization retrieval.
- You intend to train or evaluate the encoder itself — UMAP is a post-hoc dimensionality-reduction step, not a training objective.

## Inputs

- 512-dimensional ion-image representation vectors from the DeepION encoder (pretrained ResNet18, d=512)
- Preprocessed rat brain MSI matrix, DATASET/Pos_brain_data_matrix.txt (198x422 pixels, shape [X*Y, P])
- Peak list, DATASET/Pos_brain_data_peak.csv (shape [P,1])

## Outputs

- 20-dimensional dense vector per ion image (reduced-dimension representation)
- Min–max scaled features in [0, 1] for distance-based similarity measurement
- Top-5 colocalized-ion ranking per query ion (e.g., Pos_COL_result output file)

## How to apply

Take the 512-dimensional representation vectors produced by the encoder (e.g., pretrained ResNet18 backbone in DeepION) as input and apply UMAP to project them onto a 20-dimensional dense vector; the article sets the reduced dimension to 20 explicitly as a trade-off between space–time complexity and information utilization. After reduction, apply a min–max scaler to adjust all feature ranges to [0, 1] so that distance-based similarity measurement (e.g., top-5 colocalized-ion ranking in the COL-mode benchmark) is computed on comparable scales. UMAP was chosen over alternatives because it 'has been demonstrated to perform better than other dimensionality reduction methods in many fields'; the resulting 20-d vectors feed the leave-one-out linear-classifier evaluation on the 98-image, 17-category rat brain benchmark. Run UMAP after the contrastive training loop (Adam, lr 0.0003, 100 iterations) and before similarity retrieval; in the DeepION pipeline this happens inside run.py before writing the top-5 colocalized-ion results.

## Related tools

- **UMAP** (Dimensionality reduction of the 512-d encoder output to a 20-d dense vector for downstream similarity measurement (umap 0.5.1 per README requirements).)
- **DeepION** (Upstream SimSiam-based contrastive model whose COL-mode encoder produces the 512-d ion-image representations that UMAP reduces.) — https://github.com/gankLei-X/DeepION
- **scikit-learn** (Provides the min–max scaler that rescales the 20-d UMAP output features to [0, 1] before distance computation.)

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- The reduced representation must have exactly 20 dimensions per ion image (the paper's chosen trade-off dimension).
- All 20-d features are min–max scaled to [0, 1] before distance-based similarity retrieval.
- On the 98-image, 17-category rat brain benchmark, top-5 colocalized-ion classification accuracy from the UMAP-reduced COL-mode embeddings should reach approximately 86.73% with leave-one-out validation.
- Reduced-embedding retrieval should outperform the second-best SimSiam baseline by about 25 percentage points and beat PCA, t-SNE, and raw-UMAP baselines on the same data.
- The umap dependency version in the environment matches the README requirement (umap == 0.5.1).

## Limitations

- The reduced dimension of 20 was set 'by a trade-off between space–time complexity and information utilization'; other dimensionalities are not validated in this study.
- UMAP-reduced retrieval quality depends entirely on the upstream encoder: with SimSiam's natural-image-only augmentation, accuracy drops ~25 points below DeepION's COL mode, so poor representations cannot be fixed by the reduction step.
- The paper's benchmark results are specific to rat brain MSI sections (198x422 pixels, positive ion mode); transfer to other tissues or ion modes is not demonstrated here.

## Evidence

- [methods] UMAP selected over alternatives: "Since the UMAP algorithm has been demonstrated to perform better than other dimensionality reduction methods in many fields, we adopt it"
- [methods] 20-dimension trade-off: "the reduced dimension is set to 20 by a trade-off between space–time complexity and information utilization"
- [methods] min–max scaling: "a min–max scaler is adopted to adjust the range of features to [0, 1]"
- [readme] encoder output dimension: "propagated through a pair of ResNet18-based encoders that shared parameters, then output two 512-dimensional representation vectors"
- [readme] reduction module purpose: "Dimensionality Reduction module. This module is applied to further reduce the dimensionality of ion image representation to a 20-dimensional vector O for downstream tasks."
