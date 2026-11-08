---
name: stratified-cross-validation-experiment-design
description: Use when comparing classifier variants (e.g., MSMCE-integrated vs. baseline
  ResNet50) on a labeled MS dataset and you need fair fold-shared training, fixed
  hold-out evaluation, and statistical significance (p < 0.05) for accuracy/F1 differences.
when_to_use_negative:
- Ablation-study variant comparisons, where the article's protocol trains on the entire
  training set (not k-fold split) and evaluates only on the independent hold-out test
  set
- Datasets with severely imbalanced class distributions or very small sample sizes,
  where the authors warn deep models may fail to learn embedding representations
- Transfer-learning scenarios, which the authors explicitly note were not evaluated
  and are out of scope
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: PyTorch
  role: Trains the MSMCE-integrated and baseline deep classifiers end-to-end under
    the k-fold protocol
  repo: https://github.com/WoodFY/MSMCE
- name: scikit-learn
  role: Provides stratified k-fold splitting, traditional ML baselines (SVM, logistic
    regression, Random Forest), and metrics
- name: MSMCE (WoodFY/MSMCE repository)
  role: Supplies exp_multi_channel_embedding.py implementing the 6-fold stratified
    CV training/evaluation loop with early stopping
  repo: https://github.com/WoodFY/MSMCE
- name: XGBoost
  role: Gradient-boosted tree baseline trained on the same folds and test split
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:d086fa6fe02ba4c19db9157e1c51896edf7917d684c2ed237ee9373f1aeac1ae
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Stratified cross-validation experiment design

## Summary

A rigorous benchmarking protocol for comparing deep-learning and traditional ML classifiers on raw mass spectrometry data, combining stratified k-fold cross-validation (K=6) with a fixed hold-out test set and paired two-sided Wilcoxon signed-rank significance testing.

## When to use

Use when comparing classifier variants (e.g., MSMCE-integrated vs. baseline ResNet50) on a labeled MS dataset and you need fair fold-shared training, fixed hold-out evaluation, and statistical significance (p < 0.05) for accuracy/F1 differences.

## When NOT to use

- Ablation-study variant comparisons, where the article's protocol trains on the entire training set (not k-fold split) and evaluates only on the independent hold-out test set
- Datasets with severely imbalanced class distributions or very small sample sizes, where the authors warn deep models may fail to learn embedding representations
- Transfer-learning scenarios, which the authors explicitly note were not evaluated and are out of scope

## Inputs

- Pre-processed, TIC-normalized, binned mass spectrometry dataset files in datasets/ (e.g., NSCLC at spectrum_dim=12000, num_classes=2)
- Model configuration: model_name (ResNet50, DenseNet121, EfficientNetB0, LSTM, Transformer, MultiChannelEmbedding~), k_folds=6, batch_size, epochs, embedding_channels/Dim
- Baseline classifier set: SVM, logistic regression, Random Forest, XGBoost (default parameters)

## Outputs

- Per-model Accuracy and macro-averaged F1-Score on the fixed hold-out test set per fold
- Paired two-sided Wilcoxon signed-rank p-values (N=6 folds) comparing MSMCE-integrated vs. baseline models
- Aggregated results table (e.g., nsclc_results_table.csv)
- Trained model checkpoints in the save_dir

## How to apply

Load the pre-processed, TIC-normalized, binned MS dataset (e.g., NSCLC binned to 12000 m/z features) from datasets/ and apply a stratified file-level 90/10 train/hold-out split. Run 6-fold stratified cross-validation on the training set with a fixed random seed (default 3407 per the README) so all compared models see identical folds. Train each model (MSMCE-ResNet50 with embedding_channels=256, embedding_dim=1024; single-channel baselines; SVM, logistic regression, Random Forest, XGBoost with default parameters) under identical settings: Adam (lr=1e-3, weight_decay=1e-5), batch_size=64, up to 64 epochs, ReduceLROnPlateau (decay 0.1 after 5 stagnant epochs), class-weighted cross-entropy, and early stopping with patience 10. Evaluate every model on the same fixed hold-out test set, computing Accuracy and macro-averaged F1. Assess significance with the paired two-sided Wilcoxon signed-rank test across the N=6 folds, treating p < 0.05 as significant, then assemble per-model metrics and p-values into a results table (e.g., nsclc_results_table.csv).

## Related tools

- **PyTorch** (Trains the MSMCE-integrated and baseline deep classifiers end-to-end under the k-fold protocol) — https://github.com/WoodFY/MSMCE
- **scikit-learn** (Provides stratified k-fold splitting, traditional ML baselines (SVM, logistic regression, Random Forest), and metrics)
- **MSMCE (WoodFY/MSMCE repository)** (Supplies exp_multi_channel_embedding.py implementing the 6-fold stratified CV training/evaluation loop with early stopping) — https://github.com/WoodFY/MSMCE
- **XGBoost** (Gradient-boosted tree baseline trained on the same folds and test split)

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset nsclc --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 12000 --embedding_channels 256 --embedding_dim 1024 --device cuda:0 --batch_size 64 --epochs 64 --use_normalization
```

## Evaluation signals

- MSMCE-ResNet50 hold-out accuracy on NSCLC should approximate 0.9785 vs. the ResNet50 baseline 0.9661 (+1.28%, Wilcoxon p = 0.03, N = 6)
- Random Forest F1-Score on NSCLC should be around the reported 0.9935 (highest among tested models)
- All compared models must be trained and evaluated on identical stratified fold splits and the same fixed hold-out test set
- Wilcoxon p-values must come from a two-sided paired test over N = 6 fold-level metric pairs with p < 0.05 threshold
- Re-running with the fixed random_seed (default 3407) should reproduce the same fold splits and comparable metrics

## Limitations

- Embedding dimensions and parameter optimization require dataset-specific tuning; no adaptive feature selection mechanism
- Performance may be limited in extreme cases such as severely imbalanced class distributions; deep models may fail to learn embedding representations when sample size is too small
- Statistical power is constrained by K=6 folds (Wilcoxon N=6), and conclusions do not address biological interpretability of learned features or transfer across datasets

## Evidence

- [results] Statistical significance assessed with the paired Wilcoxon signed-rank test comparing models integrated with MSMCE against baselines (two-sided, N = 6 folds, p < 0.05 considered significant): "we employed the paired Wilcoxon signed-rank test to compare models integrated with MSMCE against their respective baselines"
- [results] MSMCE-ResNet50 NSCLC accuracy improvement: "MSMCE-ResNet50 achieved an accuracy of 0.9785, a statistically significant increase of 1.28% (p = 0.03, N = 6) compared to the original model accuracy of 0.9661"
- [supplementary] k-fold CV on hold-out test set: "Bootstrap 95% confidence intervals for mean performance metrics (Accuracy, Precision, Recall, F1-Score) from k-fold cross-validation on the hold-out test set for all evaluated models."
- [readme] README training command with k_folds: "python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset nsclc --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 12000 --embedding_channels 256"
- [results] Ablation protocol differs from k-fold benchmarking: "model variants were trained on the entire training set (not subjected to k-fold splitting) and their performance was subsequently evaluated on the independent hold-out test set"
