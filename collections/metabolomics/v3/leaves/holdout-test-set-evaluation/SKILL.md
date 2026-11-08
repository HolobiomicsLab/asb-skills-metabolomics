---
name: holdout-test-set-evaluation
description: Use when running an ablation study where model variants must be trained
  on the full training set (no k-fold splitting) and compared on the same independent
  hold-out test set to quantify incremental gains from each module.
when_to_use_negative:
- When the goal is statistical significance testing of MSMCE against baselines across
  data splits — use the 6-fold cross-validation with paired Wilcoxon signed-rank test
  instead
- When estimating peak GPU memory footprint or FLOPs (use thop / torchinfo profiling,
  not hold-out accuracy)
- When sample size is too small for a reliable 10% hold-out (the article notes deep
  models may fail to learn embeddings with very limited data)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3524
tools:
- name: thop
  role: Used post-hoc to profile FLOPs of the trained variants (complementary efficiency
    metrics, not part of the hold-out accuracy/F1 computation itself)
- name: torchinfo
  role: Summarizes model size/parameters of each ablation variant alongside hold-out
    performance
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:482bb7b40754804c8499a4e934057f2352abeebd17f69569fab4066648ac653b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - thop
  - torchinfo
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# holdout-test-set-evaluation

## Summary

Evaluates model variants (e.g. MSMCE-ResNet50 ablations) on an independent hold-out test set rather than cross-validation folds, to isolate the contribution of each architectural component under identical training conditions.

## When to use

Use when running an ablation study where model variants must be trained on the full training set (no k-fold splitting) and compared on the same independent hold-out test set to quantify incremental gains from each module.

## When NOT to use

- When the goal is statistical significance testing of MSMCE against baselines across data splits — use the 6-fold cross-validation with paired Wilcoxon signed-rank test instead
- When estimating peak GPU memory footprint or FLOPs (use thop / torchinfo profiling, not hold-out accuracy)
- When sample size is too small for a reliable 10% hold-out (the article notes deep models may fail to learn embeddings with very limited data)

## Inputs

- TIC-normalized MS spectrum vectors (e.g. Canine Sarcoma, spectrum_dim 15000, 12 classes, positive ion mode)
- Stratified 90%/10% file-level train/hold-out split definition
- Trained checkpoint for each ablation variant (Baseline ResNet50, +Encoder, +Encoder+Channel Embedding, full MSMCE at 256 embedding channels / 1024 embedding dim)

## Outputs

- Per-variant Accuracy and macro-averaged F1-Score on the hold-out test set (compiled into ablation_results.csv)
- Ordered ablation table showing progressive gains from Encoder → Channel Embedding → Channel Concatenation

## How to apply

Split the dataset once with a stratified 90%/10% train/hold-out split (e.g. the 12-class Canine Sarcoma TIC-normalized spectra, spectrum_dim 15000). Train each ablation variant (Baseline ResNet50, +Encoder, +Encoder+Channel Embedding, full MSMCE) on the entire training set for up to 64 epochs with Adam (lr 1e-3, weight decay 1e-5), class-weighted cross-entropy, ReduceLROnPlateau (patience 5, factor 0.1) and early stopping patience 10. Evaluate each trained variant on the same hold-out test set, computing Accuracy and macro-averaged F1-Score. Rationale: using a single untouched test set avoids fold-to-fold variance that would obscure small but real component contributions; this contrasts with the 6-fold Wilcoxon protocol used for the main MSMCE-vs-baseline comparisons (N=6, p<0.05).

## Related tools

- **thop** (Used post-hoc to profile FLOPs of the trained variants (complementary efficiency metrics, not part of the hold-out accuracy/F1 computation itself))
- **torchinfo** (Summarizes model size/parameters of each ablation variant alongside hold-out performance)

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset canine_sarcoma_posion --num_classes 12 --spectrum_dim 15000 --embedding_channels 256 --embedding_dim 1024 --batch_size 32 --epochs 64 --use_normalization --use_early_stopping
```

## Evaluation signals

- Hold-out Accuracy for the Encoder+Channel Embedding variant should approximate 0.8834 with F1 ≈ 0.8888
- Full MSMCE variant must exceed the Encoder+Channel Embedding variant (paper reports Accuracy 0.9043 for MSMCE-ResNet50)
- Rows in ablation_results.csv must show monotonically non-decreasing Accuracy from Baseline → Encoder → Encoder+Channel Embedding → full MSMCE
- Identical training hyperparameters (Adam lr 1e-3, wd 1e-5, batch_size 32, epochs ≤64, early-stop patience 10) across all variants
- Test set must remain untouched during variant selection — no metric peeking before final evaluation

## Limitations

- Requires a large enough dataset that a 10% hold-out is statistically meaningful; deep models may fail to learn embeddings when sample size is too small
- Performance may be limited in severely imbalanced class distributions, affecting hold-out metric reliability
- Embedding dimensions and parameter optimization require dataset-specific tuning; results may not transfer without retuning

## Evidence

- [results] Ablation variants trained on the entire training set (not k-fold split) and evaluated on an independent hold-out test set: "model variants were trained on the entire training set (not subjected to k-fold splitting) and their performance was subsequently evaluated on the independent hold-out test set"
- [results] Statistical significance assessed with the paired Wilcoxon signed-rank test comparing models integrated with MSMCE against baselines (two-sided, N = 6 folds, p < 0.05 considered significant): "we employed the paired Wilcoxon signed-rank test to compare models integrated with MSMCE against their respective baselines"
- [readme] README documents 64-epoch training with early stopping for the Canine Sarcoma configuration: "--dataset canine_sarcoma_posion --num_classes 12 --k_folds 6 --in_channels 1 --spectrum_dim 15000 --embedding_channels 256 --embedding_dim 1024 --device cuda:0 --batch_size 32 --epochs 64"
