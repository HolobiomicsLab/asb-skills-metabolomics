---
name: lc-ms-classification-model-training
description: Use when raw, high-dimensional MS vectors (e.g., binned to 12000 m/z
  features, TIC-normalized) must be classified end-to-end with a learnable representation
  module and compared against baselines via cross-validation.
when_to_use_negative:
- Severely imbalanced class distributions — the article notes MSMCE performance 'may
  still be limited in extreme cases, such as severely imbalanced class distributions'
- Very small sample sizes where deep models may fail to learn embedding representations,
  per the article's discussion
- Input is already a curated feature table suited to default-parameter traditional
  ML (SVM, logistic regression, Random Forest, XGBoost) without deep representation
  learning
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MSMCE (WoodFY/MSMCE)
  role: Multi-Channel Embedding Representation Module integrated with ResNet50 for
    end-to-end classification of raw MS data; training run via exp_multi_channel_embedding.py
  repo: https://github.com/WoodFY/MSMCE
- name: PyTorch
  role: Deep learning framework for training MSMCE-integrated CNN classifiers end-to-end
  repo: https://github.com/WoodFY/MSMCE
- name: scikit-learn
  role: Stratified k-fold splitting and traditional ML benchmarks (SVM, logistic regression,
    Random Forest)
  repo: https://github.com/WoodFY/MSMCE
- name: XGBoost
  role: Traditional ML benchmark classifier trained on the same folds and test split
    with default parameters
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:bdf36accfe807fcef0f66e705f0b59142d62a706897d2134974ec68da4643fd2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - MSMCE (WoodFY/MSMCE repository)
  - MSMCE (WoodFY/MSMCE)
  - PyTorch
  - XGBoost
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# lc-ms-classification-model-training

## Summary

Trains deep learning classifiers (ResNet50/DenseNet121/LSTM/Transformer) on TIC-normalized, binned raw mass spectrometry data with the MSMCE multi-channel embedding module, using stratified k-fold cross-validation, early stopping, and a fixed hold-out test set to benchmark against traditional ML baselines.

## When to use

Use when raw, high-dimensional MS vectors (e.g., binned to 12000 m/z features, TIC-normalized) must be classified end-to-end with a learnable representation module and compared against baselines via cross-validation.

## When NOT to use

- Severely imbalanced class distributions — the article notes MSMCE performance 'may still be limited in extreme cases, such as severely imbalanced class distributions'
- Very small sample sizes where deep models may fail to learn embedding representations, per the article's discussion
- Input is already a curated feature table suited to default-parameter traditional ML (SVM, logistic regression, Random Forest, XGBoost) without deep representation learning

## Inputs

- Pre-processed NSCLC dataset files in datasets/ (TIC-normalized, binned to 12000 m/z spectrum_dim)
- Training script exp_multi_channel_embedding.py from the WoodFY/MSMCE repository
- Hyperparameter configuration: model_name, num_classes, k_folds, embedding_channels, embedding_dim, batch_size, epochs

## Outputs

- Trained MSMCE-integrated and baseline model checkpoints saved to save_dir
- Per-model Accuracy and macro-averaged F1-Score on the hold-out test set per fold
- Results table (e.g., nsclc_results_table.csv) with metrics and paired Wilcoxon signed-rank p-values

## How to apply

Place processed dataset files in datasets/ (TIC-normalized, binned to 12000 m/z features for NSCLC) and split stratified at file level into 90% training / 10% hold-out test. Run exp_multi_channel_embedding.py with 6-fold stratified cross-validation (k_folds 6, random_seed 3407), embedding_channels=256, embedding_dim=1024, spectrum_dim=12000, batch_size=64, max 64 epochs, Adam (lr=1e-3, weight_decay=1e-5), ReduceLROnPlateau (decay 0.1 after 5 stagnant epochs), class-weighted cross-entropy, and early stopping with patience 10. Train the single-channel ResNet50 baseline and traditional ML benchmarks (SVM, logistic regression, Random Forest, XGBoost with default parameters) on the same folds and test split. Evaluate every model on the fixed hold-out test set computing Accuracy and macro-averaged F1-Score, then apply the paired Wilcoxon signed-rank test (two-sided, N=6 folds, p<0.05) comparing MSMCE-integrated vs. baseline models. Assemble per-model metrics and p-values into a results table (e.g., nsclc_results_table.csv).

## Related tools

- **MSMCE (WoodFY/MSMCE)** (Multi-Channel Embedding Representation Module integrated with ResNet50 for end-to-end classification of raw MS data; training run via exp_multi_channel_embedding.py) — https://github.com/WoodFY/MSMCE
- **PyTorch** (Deep learning framework for training MSMCE-integrated CNN classifiers end-to-end) — https://github.com/WoodFY/MSMCE
- **scikit-learn** (Stratified k-fold splitting and traditional ML benchmarks (SVM, logistic regression, Random Forest)) — https://github.com/WoodFY/MSMCE
- **XGBoost** (Traditional ML benchmark classifier trained on the same folds and test split with default parameters)

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset nsclc --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 12000 --embedding_channels 256 --embedding_dim 1024 --device cuda:0 --batch_size 64 --epochs 64 --use_normalization
```

## Evaluation signals

- MSMCE-ResNet50 accuracy on NSCLC should approximate 0.9785 vs. baseline 0.9661 (+1.28%) with paired Wilcoxon signed-rank p = 0.03 (N = 6)
- Random Forest F1-Score on NSCLC should fall around the reported 0.9935, the highest among tested models
- Both Accuracy and macro-averaged F1-Score are computed on the fixed hold-out test set for every model, not on CV training folds
- Statistical significance claims require two-sided paired Wilcoxon signed-rank test with p < 0.05 over N = 6 folds
- Bootstrap 95% confidence intervals for mean Accuracy, Precision, Recall, F1-Score should be reported for all evaluated models

## Limitations

- Embedding dimensions and parameter optimization require dataset-specific tuning; no adaptive feature selection mechanism
- No biological/clinical interpretability of learned embedding features (e.g., which m/z regions contribute) was investigated
- Performance may be limited under severely imbalanced class distributions or very small sample sizes
- Transfer learning / cross-dataset transferability of the module not yet evaluated

## Evidence

- [results] MSMCE-ResNet50 achieved an accuracy of 0.9785 on the NSCLC dataset, a statistically significant increase of 1.28% (p = 0.03, N = 6): "on the NSCLC dataset, MSMCE-ResNet50 achieved an accuracy of 0.9785"
- [results] we employed the paired Wilcoxon signed-rank test to compare models integrated with MSMCE against their respective baselines: "we employed the paired Wilcoxon signed-rank test to compare models integrated with MSMCE against their respective baselines"
- [readme] README usage command for NSCLC training: "python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset nsclc --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 12000 --embedding_channels 256"
- [readme] README data preparation note: "The required data files should be put into `datasets/` folder. Note that the input of each dataset is tic normalized in this implementation."
