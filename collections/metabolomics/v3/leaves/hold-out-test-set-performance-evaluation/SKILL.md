---
name: hold-out-test-set-performance-evaluation
description: Use when comparing model variants on a small, tic-normalized mass spectrometry
  feature matrix (e.g., spectrum_dim=12000, 2 classes) and you need unbiased per-model
  Accuracy/F1 plus fold-level significance testing against baselines.
when_to_use_negative:
- Ablation study variants, which the paper trains on the entire training set (not
  k-fold split) rather than the fold-wise hold-out protocol
- Datasets with severely imbalanced class distributions or very small sample sizes,
  where the authors warn deep models may fail to learn embedding representations
- Estimating model size or FLOPs — use thop/torchinfo profiling instead of performance
  evaluation
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scikit-learn
  role: stratified k-fold splitting, SVM/logistic regression/Random Forest/XGBoost
    benchmark classifiers, and metric computation
  repo: https://github.com/WoodFY/MSMCE
- name: PyTorch
  role: training MSMCE-integrated deep models (ResNet50/DenseNet121) with Adam optimizer,
    ReduceLROnPlateau scheduler, and early stopping
  repo: https://github.com/WoodFY/MSMCE
- name: Wilcoxon signed-rank test
  role: two-sided paired test (N=6 folds, p<0.05) comparing MSMCE-integrated models
    against their baselines
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:d4de9dcb97eba05e389a9a94f49e000eb154a3b445707d8bd1702f14643e3f2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - Adam optimizer
  - PyTorch
  - ReduceLROnPlateau
  - Wilcoxon signed-rank test
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# hold-out-test-set-performance-evaluation

## Summary

Evaluates each classifier (MSMCE-integrated and baseline) on a fixed, stratified 10% hold-out test set after 6-fold cross-validation, reporting Accuracy and macro-averaged F1-Score with paired Wilcoxon signed-rank significance testing, as used in the MSMCE raw mass spectrometry classification study.

## When to use

Use when comparing model variants on a small, tic-normalized mass spectrometry feature matrix (e.g., spectrum_dim=12000, 2 classes) and you need unbiased per-model Accuracy/F1 plus fold-level significance testing against baselines.

## When NOT to use

- Ablation study variants, which the paper trains on the entire training set (not k-fold split) rather than the fold-wise hold-out protocol
- Datasets with severely imbalanced class distributions or very small sample sizes, where the authors warn deep models may fail to learn embedding representations
- Estimating model size or FLOPs — use thop/torchinfo profiling instead of performance evaluation

## Inputs

- TIC-normalized CRLM feature matrix (spectrum_dim=12000, 2 classes) produced by m/z binning at 0.1 Da and 10-second RT window aggregation
- file list / sample labels for stratified 90/10 split
- trained model checkpoints or fitted models (MSMCE-integrated and baseline classifiers)
- traditional ML benchmark predictions (SVM, logistic regression, Random Forest, XGBoost with default scikit-learn settings)

## Outputs

- per-model Accuracy and macro-averaged F1-Score per fold on the fixed 10% hold-out test set
- CSV comparison table in the format of the paper's Tables 2-4
- paired Wilcoxon signed-rank test results (two-sided, N=6, p-values) for MSMCE vs baseline
- optional bootstrap 95% confidence intervals for mean Accuracy/Precision/Recall/F1

## How to apply

Partition the file list with stratified sampling into 90% training and 10% hold-out test sets, applying TIC normalization to both feature matrices after the split. Run 6-fold stratified cross-validation on the training portion for each model (e.g., MSMCE-DenseNet121 vs DenseNet121, embedding_channels=256, embedding_dim=1024, Adam lr=1e-3, weight_decay=1e-5, batch_size=64, max 64 epochs, early stopping patience=10, ReduceLROnPlateau). Evaluate every trained model on the same fixed 10% hold-out set, computing Accuracy and macro-averaged F1-Score per fold. Compare MSMCE-integrated models against their baselines with the two-sided paired Wilcoxon signed-rank test (N=6 folds, p<0.05 significant) and compile per-model results into a single CSV table matching the paper's Tables 2-4 format. Optionally report bootstrap 95% confidence intervals for the mean metrics on the hold-out set.

## Related tools

- **scikit-learn** (stratified k-fold splitting, SVM/logistic regression/Random Forest/XGBoost benchmark classifiers, and metric computation) — https://github.com/WoodFY/MSMCE
- **PyTorch** (training MSMCE-integrated deep models (ResNet50/DenseNet121) with Adam optimizer, ReduceLROnPlateau scheduler, and early stopping) — https://github.com/WoodFY/MSMCE
- **Wilcoxon signed-rank test** (two-sided paired test (N=6 folds, p<0.05) comparing MSMCE-integrated models against their baselines)

## Examples

```
python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset crlm --num_classes 2 --k_folds 6 --in_channels 1 --spectrum_dim 12000 --embedding_channels 256 --embedding_dim 1024 --device cuda:0 --batch_size 64 --epochs 64 --use_normalization
```

## Evaluation signals

- Every evaluated model has Accuracy and F1 entries on CRLM in the compiled CSV table (Tables 2-4 format)
- MSMCE-integrated models show statistically significant improvement over their baselines (p<0.05, two-sided paired Wilcoxon, N=6), e.g., MSMCE-DenseNet121 accuracy +6.66% (p=0.03) on CRLM
- The same fixed 10% hold-out set is used for all models, and both train and test feature matrices are TIC-normalized
- Per-fold metrics are computed over exactly 6 folds (k_folds=6, random_seed=3407 for reproducibility)
- Sanity anchor: DenseNet121 F1 on RCC rises from 0.6946 to 0.7952 with MSMCE in the equivalent RCC comparison

## Limitations

- Embedding dimensions and parameter optimization require dataset-specific tuning; no adaptive feature selection mechanism
- Performance may be limited in extreme cases such as severely imbalanced class distributions; deep models may fail to learn embeddings with too few samples
- Lack of biological/clinical interpretability of learned embedding features (which m/z regions contribute to classification) not investigated
- Transfer learning scenarios and cross-dataset transferability not yet evaluated

## Evidence

- [results] Wilcoxon significance protocol: "we employed the paired Wilcoxon signed-rank test to compare models integrated with MSMCE against their respective baselines"
- [results] hold-out evaluation for ablations: "model variants were trained on the entire training set (not subjected to k-fold splitting) and their performance was subsequently evaluated on the independent hold-out test set"
- [supplementary] bootstrap CIs on hold-out metrics: "Bootstrap 95% confidence intervals for mean performance metrics (Accuracy, Precision, Recall, F1-Score) from k-fold cross-validation on the hold-out test set for all evaluated models."
- [readme] TIC-normalized inputs: "The required data files should be put into `datasets/` folder. Note that the input of each dataset is tic normalized in this implementation."
- [results] CRLM accuracy improvement anchor: "on the CRLM dataset, the accuracy of MSMCE-DenseNet121 improved by 6.66%"
