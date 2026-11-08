---
name: progressive-module-ablation-training
description: Use when a multi-component representation module (e.g. Encoder + Channel
  Embedding + Channel Concatenation) needs a quantitative per-component contribution
  analysis on raw TIC-normalized mass spectrometry spectra.
when_to_use_negative:
- When the main question is cross-dataset generalization or statistical significance
  against baselines — the paper's main experiments use 6-fold cross-validation with
  paired Wilcoxon signed-rank tests (N = 6, p < 0.05), whereas this skill is the single-split
  ablation protocol
- When the model has no multi-component structure to decompose (e.g. a standalone
  Random Forest baseline) — there is nothing to ablate incrementally
- When sample size is too small for a reliable hold-out split; the paper warns deep
  models may fail to learn embeddings with insufficient samples or severely imbalanced
  class distributions
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: thop
  role: Profile FLOPs of each ablation variant to accompany accuracy gains with computational
    cost comparisons
- name: torchinfo
  role: Summarize parameter counts and model architecture for each ablation configuration
- name: PyTorch (exp_multi_channel_embedding.py)
  role: Training harness that instantiates ResNet50-based ablation variants and runs
    the 64-epoch training loop with early stopping
  repo: https://github.com/WoodFY/MSMCE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:fbbd93082af747f8679d1fda52120cfece1fd7a9212c49649b33e039ae24df65
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - PyTorch (exp_multi_channel_embedding.py)
  - thop
  - torchinfo
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# progressive-module-ablation-training

## Summary

Trains a baseline deep classifier and progressively enriched variants (baseline → baseline+Encoder → +Channel Embedding → full module) on a single train/hold-out split to quantify each component's incremental contribution, compiling Accuracy and macro-F1 into an ablation table.

## When to use

Use when a multi-component representation module (e.g. Encoder + Channel Embedding + Channel Concatenation) needs a quantitative per-component contribution analysis on raw TIC-normalized mass spectrometry spectra.

## When NOT to use

- When the main question is cross-dataset generalization or statistical significance against baselines — the paper's main experiments use 6-fold cross-validation with paired Wilcoxon signed-rank tests (N = 6, p < 0.05), whereas this skill is the single-split ablation protocol
- When the model has no multi-component structure to decompose (e.g. a standalone Random Forest baseline) — there is nothing to ablate incrementally
- When sample size is too small for a reliable hold-out split; the paper warns deep models may fail to learn embeddings with insufficient samples or severely imbalanced class distributions

## Inputs

- TIC-normalized raw mass spectrometry spectrum vectors (e.g. Canine Sarcoma 12-class dataset, spectrum_dim 15000, positive ion mode)
- Stratified file-level 90%/10% train/hold-out split
- Baseline classifier checkpoints (e.g. ResNet50) and module component definitions (Encoder FCN, 1D-conv Channel Embedding, channel concatenation)

## Outputs

- ablation_results.csv with per-variant Accuracy and macro F1-Score rows (Baseline, +Encoder, +Channel Embedding, full MSMCE)
- Trained checkpoints for each ablation variant
- Verified monotonic (or near-monotonic) performance progression across components

## How to apply

Start from the baseline classifier (e.g. ResNet50) on TIC-normalized spectrum vectors and build three ablation variants: (1) baseline plus a two-layer fully connected Encoder with LayerNorm, ReLU and Dropout; (2) add Channel Embedding implemented as two stacked 1D conv layers (kernel_size 3, batch normalization, ReLU, Dropout); (3) the full module adding channel concatenation of the global embedding E' with multi-channel embedding C2 along the channel dimension, configured at 256 embedding channels and embedding dimension 1024. Perform a stratified file-level 90%/10% train/hold-out split, train each variant on the entire training set (not k-fold, unlike the main experiments) for up to 64 epochs with Adam (lr 1e-3, weight decay 1e-5), class-weighted cross-entropy, ReduceLROnPlateau (patience 5, factor 0.1), and early stopping on validation loss with patience 10. Evaluate on the independent hold-out test set with Accuracy and macro-averaged F1-Score, then compile rows into ablation_results.csv ordered to show progressive gains. Accept the table if the Channel Embedding variant reaches approximately Accuracy 0.8834 / F1 0.8888 and the full module surpasses it (paper reports 0.9043 for MSMCE-ResNet50), demonstrating each component's incremental contribution.

## Related tools

- **thop** (Profile FLOPs of each ablation variant to accompany accuracy gains with computational cost comparisons)
- **torchinfo** (Summarize parameter counts and model architecture for each ablation configuration)
- **PyTorch (exp_multi_channel_embedding.py)** (Training harness that instantiates ResNet50-based ablation variants and runs the 64-epoch training loop with early stopping) — https://github.com/WoodFY/MSMCE

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset canine_sarcoma_posion --num_classes 12 --in_channels 1 --spectrum_dim 15000 --embedding_channels 256 --embedding_dim 1024 --batch_size 32 --epochs 64 --use_normalization --use_early_stopping --patience 10
```

## Evaluation signals

- Channel Embedding variant reaches approximately Accuracy 0.8834 / F1-Score 0.8888 on the hold-out set
- Full module (with Channel Concatenation) surpasses the Encoder+Channel Embedding variant (paper reports Accuracy 0.9043 for MSMCE-ResNet50)
- Baseline variant underperforms all enriched variants, showing each component adds incremental gain
- All variants trained with identical hyperparameters (epochs 64, Adam lr 1e-3, weight decay 1e-5, early-stopping patience 10) so gains are attributable to components, not tuning
- Metrics use macro (unweighted) averaging consistent with the paper's reporting

## Limitations

- Ablation variants were trained on the entire training set (not k-fold splitting), so single-split variance is not quantified for this comparison
- Embedding dimensions and parameter optimization require dataset-specific tuning; no adaptive mechanism exists
- Performance may be limited with severely imbalanced class distributions or very small sample sizes
- Biological/clinical interpretability of learned embedding features (which m/z regions contribute) was not investigated

## Evidence

- [results] Ablation study variants trained on the entire training set (not k-fold split) and evaluated on an independent hold-out test set: "model variants were trained on the entire training set (not subjected to k-fold splitting) and their performance was subsequently evaluated on the independent hold-out test set"
- [abstract] MSMCE module concatenation mechanism: "we implement a feature fusion mechanism by concatenating the initial encoded representation with the multi-channel embeddings along the channel dimension"
- [readme] Canine Sarcoma training command with module hyperparameters: "python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset canine_sarcoma_posion --num_classes 12 --k_folds 6 --in_channels 1 --spectrum_dim 15000 --embedding_channels"
- [results] FLOPs estimation tools used in results: "FLOPs were calculated using the profile function from the Python library thop."
