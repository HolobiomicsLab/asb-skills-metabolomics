---
name: k-fold-cross-validation-training
description: Use when training deep learning (ResNet50, DenseNet121, LSTM, Transformer)
  or traditional ML classifiers on TIC-normalized MS spectra and you need fold-level
  performance estimates plus paired statistical comparison of an augmentation module
  against baselines.
when_to_use_negative:
- 'Running ablation-study variants: per the paper, these are trained on the entire
  training set (not subjected to k-fold splitting) and evaluated only on the independent
  hold-out test set'
- Datasets with severely imbalanced class distributions or too few samples, where
  the paper notes performance may be limited and deep models may fail to learn embedding
  representations
- Data that has not been processed to the expected raw-MS feature matrix (m/z binning
  at 0.1 Da, 10-second RT window aggregation) and TIC-normalized form
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0092
tools:
- name: scikit-learn
  role: Provides stratified k-fold splitting, the traditional ML benchmarks (SVM,
    logistic regression, Random Forest, XGBoost) under default configurations, and
    metric computation on the hold-out test set
- name: PyTorch
  role: Training backend for the MSMCE-integrated and baseline deep models across
    the 6 folds (Adam optimizer, ReduceLROnPlateau scheduler, early stopping)
  repo: https://github.com/WoodFY/MSMCE
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:d6028f60fc1266568d341185803574a7c5c544c5770bac446175f1489df0dd5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - Adam optimizer
  - PyTorch
  - ReduceLROnPlateau
  - scikit-learn
  - thop
  - torchinfo
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# k-fold-cross-validation-training

## Summary

A 6-fold stratified cross-validation training and evaluation protocol for TIC-normalized raw mass spectrometry classification datasets (NSCLC, CRLM, RCC, Canine Sarcoma), designed so per-fold results support paired two-sided Wilcoxon signed-rank tests (N=6, p<0.05) between MSMCE-integrated models and their baselines.

## When to use

Use when training deep learning (ResNet50, DenseNet121, LSTM, Transformer) or traditional ML classifiers on TIC-normalized MS spectra and you need fold-level performance estimates plus paired statistical comparison of an augmentation module against baselines.

## When NOT to use

- Running ablation-study variants: per the paper, these are trained on the entire training set (not subjected to k-fold splitting) and evaluated only on the independent hold-out test set
- Datasets with severely imbalanced class distributions or too few samples, where the paper notes performance may be limited and deep models may fail to learn embedding representations
- Data that has not been processed to the expected raw-MS feature matrix (m/z binning at 0.1 Da, 10-second RT window aggregation) and TIC-normalized form

## Inputs

- TIC-normalized mass spectrometry feature matrix (e.g., spectrum_dim=12000 for NSCLC/CRLM, 9900 for RCC, 15000 for Canine Sarcoma) with class labels (2-class or 12-class)
- File list / sample identifiers for stratified 90/10 train/test partitioning
- Training configuration (k_folds=6, batch_size, epochs=64, lr=1e-3, weight_decay=1e-5, patience=10, random_seed=3407)

## Outputs

- Per-fold Accuracy and macro-averaged F1-Score for each model on the fixed 10% hold-out test set (N=6 folds)
- Paired Wilcoxon signed-rank test statistics (two-sided, N=6, p<0.05) comparing MSMCE-integrated vs baseline models
- Bootstrap 95% confidence intervals for mean Accuracy, Precision, Recall, and F1-Score
- Results table (CSV, in the format of the paper's Tables 2-5) with per-model metrics

## How to apply

First partition the dataset with stratified sampling into a 90% training set and a fixed 10% hold-out test set, then run 6-fold stratified cross-validation (k_folds=6) on the training set only. Train each fold with the Adam optimizer (lr=1e-3, weight_decay=1e-5), ReduceLROnPlateau (decay factor 0.1 after 5 stagnant epochs), early stopping (patience=10), batch_size=64 (32 for the 12-class Canine Sarcoma dataset), and a maximum of 64 epochs, using random_seed=3407 for reproducibility; keep fold splits and hyperparameters identical between MSMCE-integrated models and single-channel baselines so comparisons are paired. Evaluate every fold on the independent hold-out test set, recording Accuracy and macro-averaged F1-Score per fold. Assess significance with a two-sided paired Wilcoxon signed-rank test across the N=6 folds (p<0.05), and report Bootstrap 95% confidence intervals for mean Accuracy, Precision, Recall, and F1-Score. Exception per the paper: ablation-study variants are trained on the entire training set (not the k-fold split) and evaluated only on the hold-out test set.

## Related tools

- **scikit-learn** (Provides stratified k-fold splitting, the traditional ML benchmarks (SVM, logistic regression, Random Forest, XGBoost) under default configurations, and metric computation on the hold-out test set)
- **PyTorch** (Training backend for the MSMCE-integrated and baseline deep models across the 6 folds (Adam optimizer, ReduceLROnPlateau scheduler, early stopping)) — https://github.com/WoodFY/MSMCE

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset canine_sarcoma_posion --num_classes 12 --k_folds 6 --in_channels 1 --spectrum_dim 15000 --embedding_channels 256 --embedding_dim 1024 --device cuda:0 --batch_size 32 --epochs 64 --use_normalization
```

## Evaluation signals

- Cross-validation must use exactly k_folds=6 stratified folds on the 90% training partition, with the 10% hold-out test set never used for training
- Per-fold Accuracy and macro F1-Score must be reported for both MSMCE-integrated and baseline models under identical fold splits and hyperparameters so the paired Wilcoxon signed-rank test (two-sided, N=6) is valid
- Result magnitudes should reproduce reported findings, e.g., MSMCE-ResNet50 accuracy 0.9785 on NSCLC (+1.28%, p=0.03) and 0.9043 on Canine Sarcoma (+24.99% over baseline 0.7235, p=0.03)
- Statistical significance should hold at p<0.05 with N=6 folds (e.g., CRLM MSMCE-DenseNet121 +6.66%, p=0.03)
- Bootstrap 95% confidence intervals for mean Accuracy, Precision, Recall, and F1-Score accompany the k-fold metrics

## Limitations

- Embedding dimensions and parameter optimization require dataset-specific tuning; the protocol does not adapt k_folds or hyperparameters automatically
- Performance may be limited in extreme cases such as severely imbalanced class distributions, and deep models may fail to learn embedding representations when sample size is too small
- Cross-validation alone does not address transferability: the paper notes effectiveness in transfer learning scenarios and across datasets/tasks has not been evaluated

## Evidence

- [results] we employed the paired Wilcoxon signed-rank test to compare models integrated with MSMCE against their respective baselines: "we employed the paired Wilcoxon signed-rank test to compare models integrated with MSMCE against their respective baselines"
- [supplementary] Bootstrap 95% confidence intervals for mean performance metrics from k-fold cross-validation: "Bootstrap 95% confidence intervals for mean performance metrics (Accuracy, Precision, Recall, F1-Score) from k-fold cross-validation on the hold-out test set for all evaluated models."
- [results] Ablation variants trained on the entire training set (not k-fold split): "model variants were trained on the entire training set (not subjected to k-fold splitting) and their performance was subsequently evaluated on the independent hold-out test set"
- [readme] README documents k_folds parameter with default 6: "| k_folds            | Number of folds for cross-validation (default: `6`)."
- [readme] README canine sarcoma invocation with k_folds 6 and 12 classes: "python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset canine_sarcoma_posion --num_classes 12 --k_folds 6 --in_channels 1 --spectrum_dim 15000"
