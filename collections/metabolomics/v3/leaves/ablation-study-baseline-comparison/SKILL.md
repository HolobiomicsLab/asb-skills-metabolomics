---
name: ablation-study-baseline-comparison
description: Use when you must attribute a reported accuracy gap (e.g. DeepION COL
  86.73% vs second- SimSiam on 98 annotated ion images, 17 categories) to a specific
  design choice, by retraining the same SimSiam-based architecture with and without
  that component and comparing on a fixed benchmark.
when_to_use_negative:
- The ion images lack manual annotation, so the 98-image/17-category leave-one-out
  linear classifier benchmark cannot be constructed for a like-for-like comparison
- The target task is isotope ion discovery (ISO mode with intensity-dependent missing
  value augmentation) rather than colocalized ion (COL) representation quality
- You want to compare architecturally different methods (e.g. SIM-based PCC/R2 vs
  deep models) rather than attribute a single model's gain to one of its own components
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: DeepION
  role: Full COL-mode model whose MSI-specific augmentation (color jitter, filtering,
    Poisson noise, random missing value) is the component under ablation
  repo: https://github.com/gankLei-X/DeepION
- name: SimSiam
  role: Stripped baseline sharing DeepION's architecture and training settings but
    restricted to natural-image augmentations; DeepION is constructed on top of it
  repo: https://github.com/BioNet-XMU/DeepION
- name: Resnet18
  role: Pretrained backbone encoder (512-d representations) shared by both ablated
    and baseline models to keep architectures identical
- name: UMAP
  role: Dimensionality reduction of encoder outputs to 20-d embeddings before similarity
    retrieval in both conditions
- name: PyTorch
  role: Implementation framework in which both models are trained under matched optimizer
    and augmentation settings
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.3c05002
    title: 'DeepION: A Deep Learning-Based Low-Dimensional Representation Model of
      Ion Images for Mass Spectrometry Imaging'
  source_content_hash: sha256:04fccba99331244037e03a6851e93b5f2bc96c84585c805fff5c7ea37f3aba3c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# ablation-study-baseline-comparison

## Summary

Runs a controlled ablation in which the full model and a stripped baseline share identical architecture and training settings so that any performance delta can be attributed to a single component — here, DeepION's MSI-specific augmentation (Poisson noise, random missing value) versus SimSiam's natural-image-only augmentation on the rat brain colocalization benchmark.

## When to use

Use when you must attribute a reported accuracy gap (e.g. DeepION COL 86.73% vs second-best SimSiam on 98 annotated ion images, 17 categories) to a specific design choice, by retraining the same SimSiam-based architecture with and without that component and comparing on a fixed benchmark.

## When NOT to use

- The ion images lack manual annotation, so the 98-image/17-category leave-one-out linear classifier benchmark cannot be constructed for a like-for-like comparison
- The target task is isotope ion discovery (ISO mode with intensity-dependent missing value augmentation) rather than colocalized ion (COL) representation quality
- You want to compare architecturally different methods (e.g. SIM-based PCC/R2 vs deep models) rather than attribute a single model's gain to one of its own components

## Inputs

- Preprocessed rat brain MSI matrix (DATASET/Pos_brain_data_matrix.txt, [X*Y,P] shaped, 198x422 pixels)
- Preprocessed MSI peak list (DATASET/Pos_brain_data_peak.csv, [P,1])
- Benchmark of 98 manually annotated ion images spanning 17 categories (leave-one-out linear classifier evaluation set)

## Outputs

- augmentation_ablation_comparison.csv — side-by-side leave-one-out classification and top-5 colocalized-ion retrieval accuracies for DeepION COL vs vanilla SimSiam
- 20-dimensional UMAP embeddings of ion images for each model, min–max scaled to [0,1]

## How to apply

Train two contrastive models that are identical in every respect except the augmentation pipeline: DeepION COL mode (SimSiam architecture with pretrained Resnet18 encoder producing 512-d vectors, three-layer FC projection MLP, two-layer FC prediction MLP, stop-gradient, Adam at lr 0.0003 with beta1=0.5/beta2=0.99, N(0,0.02) weight init, 100 iterations) versus a vanilla SimSiam baseline limited to natural-image augmentations (color jitter, filtering) that omits the Poisson-noise and random-missing-value operations encoding MSI priors. Reduce each encoder's outputs to a 20-dimensional UMAP embedding (the paper's chosen trade-off between space–time complexity and information utilization) and min–max scale features to [0,1]. Evaluate identically for both models: leave-one-out linear classification on the 98 manually annotated ion images across 17 categories, plus distance-based retrieval of the top-5 colocalized ions as in the COL-mode run configuration. Record both models' accuracies side by side (e.g. augmentation_ablation_comparison.csv) so the delta isolates the augmentation strategy; the accepted outcome is DeepION COL reproducing or approximating 86.73% while exceeding SimSiam by about 25 percentage points.

## Related tools

- **DeepION** (Full COL-mode model whose MSI-specific augmentation (color jitter, filtering, Poisson noise, random missing value) is the component under ablation) — https://github.com/gankLei-X/DeepION
- **SimSiam** (Stripped baseline sharing DeepION's architecture and training settings but restricted to natural-image augmentations; DeepION is constructed on top of it) — https://github.com/BioNet-XMU/DeepION
- **Resnet18** (Pretrained backbone encoder (512-d representations) shared by both ablated and baseline models to keep architectures identical)
- **UMAP** (Dimensionality reduction of encoder outputs to 20-d embeddings before similarity retrieval in both conditions)
- **PyTorch** (Implementation framework in which both models are trained under matched optimizer and augmentation settings)

## Examples

```
python run.py --input_Matrix .../DATASET/Pos_brain_data_matrix.txt --input_PeakList .../DATASET/Pos_brain_data_peak.csv --input_shape 198 422 --mode COL --ion_mode positive --num 5 --output_file Pos_COL_result
```

## Evaluation signals

- DeepION COL mode reproduces or approximates the reported 86.73% leave-one-out classification accuracy on the 98-image, 17-category benchmark
- The accuracy gap over the vanilla SimSiam baseline is approximately 25 percentage points, confirming the augmentation strategy as the source of the advantage
- A diff check confirms the ONLY differing training configuration is the augmentation set: Poisson noise and random missing value present in DeepION, absent in SimSiam (color jitter and filtering shared)
- Both models output 512-d encoder representations reduced to 20-d UMAP embeddings, min–max scaled to [0,1], before the identical top-5 distance-based retrieval procedure
- Neither model produces collapsed solutions, verified by the stop-gradient prediction-MLP design common to both runs

## Limitations

- Batch size is constrained by GPU memory (workstation with an Nvidia GTX 2080Ti), limiting training throughput
- The attribution of the ~25-point gap to augmentation is the authors' interpretation of a single dataset (rat brain, positive ion mode); no statistical significance testing of the difference is reported
- Training uses only 100 iterations, so absolute accuracies may be sensitive to compute budget rather than purely to the augmentation strategy

## Evidence

- [results] the COL mode of the DeepION model achieves 86.73%, which outperforms the second-best SimSiam model by about 25%: "the COL mode of the DeepION model achieves 86.73%, which outperforms the second-best SimSiam model by about 25%"
- [methods] DeepION constructed on SimSiam: "The proposed DeepION model is constructed based on the SimSiam model"
- [methods] MSI augmentation operations enumerated: "including color jitter, filtering, Poisson noise, random missing value, and intensity-dependent missing value"
- [readme] README COL-mode augmentation definition: "the T_COL including color jitter, filtering, Poisson noise, and random missing value is carried in COL mode"
- [methods] Reduced dimension choice: "the reduced dimension is set to 20 by a trade-off between space–time complexity and information utilization"
