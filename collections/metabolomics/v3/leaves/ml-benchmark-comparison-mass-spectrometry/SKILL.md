---
name: ml-benchmark-comparison-mass-spectrometry
description: Use when comparing a proposed deep learning model (e.g., MSMCE-ResNet50)
  against single-channel baselines and traditional ML classifiers (SVM, logistic regression,
  Random Forest, XGBoost) on binned, TIC-normalized MS data such as NSCLC spectra.
when_to_use_negative:
- When the input is already a curated feature table from a different modality with
  no m/z-binned spectra, since the benchmark assumes raw MS vectors at a fixed spectrum_dim
- When sample sizes are too small or class distributions are severely imbalanced —
  the article notes deep models may fail to learn embedding representations and performance
  may be limited in these extreme cases
- When the goal is transfer learning or cross-dataset generalization — the article
  states transferability across datasets/tasks has not yet been evaluated
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: PyTorch
  role: Trains the MSMCE-integrated deep classifier (e.g., MultiChannelEmbeddingResNet50)
    end-to-end with class-weighted cross-entropy and early stopping
  repo: https://github.com/WoodFY/MSMCE
- name: scikit-learn
  role: Provides stratified k-fold cross-validation, the traditional ML baselines
    (SVM, logistic regression, Random Forest), and metric computation
  repo: https://github.com/WoodFY/MSMCE
- name: XGBoost
  role: Gradient-boosted tree baseline trained on the same folds and test split with
    default parameters
- name: MSMCE (WoodFY/MSMCE repository)
  role: Supplies the multi-channel embedding module and exp_multi_channel_embedding.py
    training script used for the proposed model arm of the benchmark
  repo: https://github.com/WoodFY/MSMCE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:960e4ae98812a7ed608e88b8b78f6b5418e008881d2fa6442b07597c3c061f96
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - MSMCE (WoodFY/MSMCE repository)
  - PyTorch
  - XGBoost
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# ml-benchmark-comparison-mass-spectrometry

## Summary

Benchmark a deep learning classifier against traditional ML baselines (SVM, logistic regression, Random Forest, XGBoost) on raw mass spectrometry data using stratified k-fold cross-validation and a paired Wilcoxon signed-rank test to establish statistically significant performance differences.

## When to use

Use when comparing a proposed deep learning model (e.g., MSMCE-ResNet50) against single-channel baselines and traditional ML classifiers (SVM, logistic regression, Random Forest, XGBoost) on binned, TIC-normalized MS data such as NSCLC spectra.

## When NOT to use

- When the input is already a curated feature table from a different modality with no m/z-binned spectra, since the benchmark assumes raw MS vectors at a fixed spectrum_dim
- When sample sizes are too small or class distributions are severely imbalanced — the article notes deep models may fail to learn embedding representations and performance may be limited in these extreme cases
- When the goal is transfer learning or cross-dataset generalization — the article states transferability across datasets/tasks has not yet been evaluated

## Inputs

- Pre-processed, TIC-normalized MS spectra binned to a fixed m/z feature dimension (e.g., 12000 features for NSCLC) from the datasets/ folder
- Stratified file-level train/hold-out split specification (90% training / 10% hold-out test)
- Deep learning model checkpoints/configurations (e.g., MultiChannelEmbeddingResNet50 with embedding_channels=256, embedding_dim=1024)
- Traditional ML baseline configurations (SVM, logistic regression, Random Forest, XGBoost with default parameters)

## Outputs

- Per-model Accuracy and macro-averaged F1-Score on the fixed hold-out test set
- Paired Wilcoxon signed-rank test p-values (two-sided, N=6 folds) for MSMCE-integrated vs. baseline comparisons
- Aggregated results table (e.g., nsclc_results_table.csv) with metrics and p-values per model

## How to apply

Load pre-processed, TIC-normalized spectra binned to a fixed m/z dimension (e.g., 12000 features for NSCLC) and apply a stratified 90%/10% file-level train/hold-out split. Run 6-fold stratified cross-validation (K=6) on the training set with a fixed random seed, training the deep model end-to-end (e.g., Adam lr=1e-3, weight_decay=1e-5, batch_size=64, max 64 epochs, ReduceLROnPlateau with decay 0.1 after 5 stagnant epochs, class-weighted cross-entropy, early stopping patience 10) alongside traditional ML benchmarks with default parameters on identical folds. Evaluate every model on the fixed hold-out test set computing Accuracy and macro-averaged F1-Score. Assess significance with the paired Wilcoxon signed-rank test (two-sided, N=6 folds, p < 0.05 threshold) comparing the MSMCE-integrated model against its baseline. Assemble per-model metrics and p-values into a results table (e.g., nsclc_results_table.csv).

## Related tools

- **PyTorch** (Trains the MSMCE-integrated deep classifier (e.g., MultiChannelEmbeddingResNet50) end-to-end with class-weighted cross-entropy and early stopping) — https://github.com/WoodFY/MSMCE
- **scikit-learn** (Provides stratified k-fold cross-validation, the traditional ML baselines (SVM, logistic regression, Random Forest), and metric computation) — https://github.com/WoodFY/MSMCE
- **XGBoost** (Gradient-boosted tree baseline trained on the same folds and test split with default parameters)
- **MSMCE (WoodFY/MSMCE repository)** (Supplies the multi-channel embedding module and exp_multi_channel_embedding.py training script used for the proposed model arm of the benchmark) — https://github.com/WoodFY/MSMCE

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset nsclc --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 12000 --embedding_channels 256 --embedding_dim 1024 --device cuda:0 --batch_size 64 --epochs 64 --use_normalization
```

## Evaluation signals

- MSMCE-ResNet50 hold-out accuracy should approximate the reported 0.9785 on NSCLC, versus the ResNet50 baseline at 0.9661 (+1.28%)
- Wilcoxon signed-rank test should yield p = 0.03 (two-sided, N = 6 folds), meeting the p < 0.05 significance threshold
- Random Forest F1-Score should land near the reported 0.9935, the highest among all tested models on NSCLC
- All models must be trained and evaluated on identical stratified folds and the same fixed 10% hold-out test set
- Results table must contain both Accuracy and macro-averaged F1-Score per model plus paired p-values

## Limitations

- The choice of embedding dimensions and parameter optimization requires dataset-specific tuning; no adaptive feature selection mechanism is available
- Performance may be limited in extreme cases such as severely imbalanced class distributions; deep models may fail to learn embedding representations when sample size is too small
- Biological/clinical interpretability of learned embedding features (e.g., which m/z regions contribute to classification) was not investigated
- Effectiveness in transfer learning scenarios and transferability across datasets/tasks has not been evaluated

## Evidence

- [results] MSMCE-ResNet50 achieved an accuracy of 0.9785 on the NSCLC dataset, a statistically significant increase of 1.28% (p = 0.03, N = 6) compared to the original model accuracy of 0.9661: "MSMCE-ResNet50 achieved an accuracy of 0.9785, a statistically significant increase of 1.28% (p = 0.03, N = 6) compared to the original model accuracy of 0.9661"
- [results] Wilcoxon test protocol: "we employed the paired Wilcoxon signed-rank test to compare models integrated with MSMCE against their respective baselines"
- [results] Random Forest baseline result: "the RF model achieved the highest F1-Score of 0.9935 among all tested models"
- [readme] README training command for NSCLC benchmark: "python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset nsclc --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 12000"
