---
name: macro-averaged-multiclass-metrics-computation
description: Use when scoring multi-class classifiers (e.g., 12-class Canine Sarcoma)
  on a fixed independent hold-out test set, where per-class performance must be averaged
  equally via macro-averaged Accuracy and F1-Score.
when_to_use_negative:
- The task is binary classification (e.g., the NSCLC, CRLM, or RCC 2-class datasets)
  where plain accuracy and binary F1 suffice without macro averaging across 12 classes
- Metrics are needed on k-fold validation subsets rather than the independent fixed
  hold-out test set, which would not match the paper's evaluation protocol
- Class distributions are severely imbalanced and small-sample, where the article
  notes deep models may fail to learn embeddings and macro-averaged metrics may be
  of limited use
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: scikit-learn
  role: Computes macro-averaged Accuracy and F1-Score on hold-out test predictions
    (scikit_learn 1.3.0 is the pinned dependency in the repository requirements)
- name: thop
  role: Not used for metrics computation; listed in the paper only for FLOPs profiling
    of the same models whose accuracy/F1 are tabulated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:f548d865dbadab7438fb318902af1c6a530653e830378d50584eaac47255a6e2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scikit-learn
  - thop
  - torchinfo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# macro-averaged-multiclass-metrics-computation

## Summary

Computes macro-averaged Accuracy and F1-Score for multi-class MS data classification (e.g., the 12-class Canine Sarcoma task) on an independent fixed hold-out test set, so that MSMCE-enhanced models can be compared against baselines and traditional ML benchmarks in the paper's Tables 2-5 format.

## When to use

Use when scoring multi-class classifiers (e.g., 12-class Canine Sarcoma) on a fixed independent hold-out test set, where per-class performance must be averaged equally via macro-averaged Accuracy and F1-Score.

## When NOT to use

- The task is binary classification (e.g., the NSCLC, CRLM, or RCC 2-class datasets) where plain accuracy and binary F1 suffice without macro averaging across 12 classes
- Metrics are needed on k-fold validation subsets rather than the independent fixed hold-out test set, which would not match the paper's evaluation protocol
- Class distributions are severely imbalanced and small-sample, where the article notes deep models may fail to learn embeddings and macro-averaged metrics may be of limited use

## Inputs

- Predicted class labels for the independent hold-out test set from each model (MSMCE-ResNet50, single-channel ResNet50 baseline, SVM, logistic regression, Random Forest, XGBoost)
- Ground-truth class labels for the 12-class Canine Sarcoma hold-out test set
- Per-fold predictions from k_folds=6 cross-validation for paired statistical testing

## Outputs

- Macro-averaged Accuracy per model (e.g., 0.9043 for MSMCE-ResNet50 vs 0.7235 for baseline ResNet50)
- Macro-averaged F1-Score per model
- Results table in the format of the paper's Tables 2-5
- Paired Wilcoxon signed-rank test p-values (two-sided, N = 6 folds) comparing MSMCE-enhanced vs baseline models
- Optional Bootstrap 95% confidence intervals for mean performance metrics

## How to apply

After training each model (MSMCE-ResNet50, ResNet50 baseline, and traditional ML benchmarks such as SVM, logistic regression, Random Forest, and XGBoost with default parameters) on the 6-fold splits of the TIC-normalized Canine Sarcoma dataset (spectrum_dim 15000, batch_size 32), generate predictions on the independent fixed hold-out test set rather than fold validation data. Compute macro-averaged Accuracy and macro-averaged F1-Score across all 12 classes using scikit-learn 1.3.0, treating every class equally regardless of sample count. Compile the per-model metrics into a results table matching the paper's Tables 2-5 format, and attach statistical significance from paired two-sided Wilcoxon signed-rank tests across the N = 6 folds with p < 0.05 as the significance threshold. Optionally supplement point estimates with Bootstrap 95% confidence intervals for Accuracy, Precision, Recall, and F1-Score as described in the supplementary material.

## Related tools

- **scikit-learn** (Computes macro-averaged Accuracy and F1-Score on hold-out test predictions (scikit_learn 1.3.0 is the pinned dependency in the repository requirements))
- **thop** (Not used for metrics computation; listed in the paper only for FLOPs profiling of the same models whose accuracy/F1 are tabulated)

## Examples

```
from sklearn.metrics import accuracy_score, f1_score; acc = accuracy_score(y_true, y_pred); f1_macro = f1_score(y_true, y_pred, average='macro')
```

## Evaluation signals

- MSMCE-ResNet50 accuracy on the 12-class Canine Sarcoma task should reach approximately 0.9043, a 24.99% increase over the 0.7235 single-channel ResNet50 baseline (p = 0.03, N = 6)
- Paired two-sided Wilcoxon signed-rank tests comparing MSMCE-integrated models against baselines should yield p < 0.05 for the reported improvements
- Metrics must be computed on the independent fixed hold-out test set, not on k-fold validation subsets
- Averaging must be macro (per-class F1 averaged over all 12 classes), not micro- or weighted-averaged
- Bootstrap 95% confidence intervals should accompany mean Accuracy, Precision, Recall, and F1-Score as in the supplementary tables

## Limitations

- Performance may be limited in extreme cases such as severely imbalanced class distributions, where macro-averaged metrics can be dominated by small or poorly learned classes
- Deep models may fail to learn embedding representations when sample size is too small, making hold-out metric estimates unreliable
- Point estimates require the paired Wilcoxon signed-rank test across N = 6 folds to claim statistical significance; single-split metrics alone are insufficient

## Evidence

- [results] macro metrics on 12-class task: "the RF model achieved the highest F1-Score of 0.9935 among all tested models"
- [results] wilcoxon significance testing: "we employed the paired Wilcoxon signed-rank test to compare models integrated with MSMCE against their respective baselines"
- [results] wilcoxon threshold detail: [evidence span withheld — anchor status: not_found]
- [supplementary] hold-out evaluation protocol: "Bootstrap 95% confidence intervals for mean performance metrics (Accuracy, Precision, Recall, F1-Score) from k-fold cross-validation on the hold-out test set for all evaluated models."
- [readme] canine sarcoma training command grounding the 12-class setup: "python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset canine_sarcoma_posion --num_classes 12 --k_folds 6 --in_channels 1 --spectrum_dim 15000"
