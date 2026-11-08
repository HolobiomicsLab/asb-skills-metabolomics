---
name: paired-wilcoxon-significance-testing
description: Use when comparing paired per-fold performance metrics (Accuracy or macro
  F1-Score) of a MSMCE-integrated model vs. its baseline from 6-fold stratified cross-validation
  to establish statistical significance.
when_to_use_negative:
- When only a single train/test split (no k-fold or repeated paired runs) is available,
  since the test requires paired observations and the paper's protocol uses N = 6
  folds
- When comparing more than two models or unpaired model families (e.g. MSMCE-ResNet50
  vs a different architecture's folds) rather than one model vs its own MSMCE-integrated
  counterpart
- When the comparison target is a traditional ML benchmark (SVM, logistic regression,
  Random Forest, XGBoost) evaluated without matched-fold pairing to the deep model
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: scikit-learn
  role: Provides cross-validation and metric computation (Accuracy, macro F1) that
    produce the paired fold-level scores fed into the Wilcoxon test
- name: PyTorch
  role: Trains the paired MSMCE-integrated and baseline models (ResNet50/DenseNet121)
    over identical 6-fold splits so fold-level outputs are matched
  repo: https://github.com/WoodFY/MSMCE
- name: Wilcoxon signed-rank test
  role: Two-sided paired significance test applied to the N = 6 fold pairs to compare
    MSMCE-integrated models against their baselines at p < 0.05
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:997332bb3d5a7a625ef8f88b665eec243ef34b61afdbd9941b20767616519399
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
  - thop
  - torchinfo
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# paired-wilcoxon-significance-testing

## Summary

A statistical significance procedure that applies the paired Wilcoxon signed-rank test (two-sided, N = 6 folds, p < 0.05 threshold) to compare models integrated with the MSMCE module against their respective baselines on mass spectrometry classification tasks.

## When to use

Use when comparing paired per-fold performance metrics (Accuracy or macro F1-Score) of a MSMCE-integrated model vs. its baseline from 6-fold stratified cross-validation to establish statistical significance.

## When NOT to use

- When only a single train/test split (no k-fold or repeated paired runs) is available, since the test requires paired observations and the paper's protocol uses N = 6 folds
- When comparing more than two models or unpaired model families (e.g. MSMCE-ResNet50 vs a different architecture's folds) rather than one model vs its own MSMCE-integrated counterpart
- When the comparison target is a traditional ML benchmark (SVM, logistic regression, Random Forest, XGBoost) evaluated without matched-fold pairing to the deep model

## Inputs

- Per-fold Accuracy and macro-averaged F1-Score arrays (N = 6 folds) for the MSMCE-integrated model
- Per-fold Accuracy and macro-averaged F1-Score arrays (N = 6 folds) for the matched baseline model trained on identical fold splits
- Model names and dataset identifiers (NSCLC, CRLM, RCC, Canine Sarcoma) for reporting

## Outputs

- Two-sided Wilcoxon signed-rank p-value per model-vs-baseline comparison (e.g. p = 0.03)
- Statistical significance verdict at p < 0.05 for each MSMCE vs baseline pairing
- Significance annotations for the results tables (Tables 2-5) comparing MSMCE-augmented and baseline classifiers

## How to apply

Run both the MSMCE-integrated model (e.g. MultiChannelEmbeddingResNet50 with embedding_channels 256, embedding_dim 1024) and its baseline (e.g. single-channel ResNet50) over the same 6-fold stratified cross-validation splits so that fold-level scores are paired. Collect per-fold Accuracy and macro-averaged F1-Score on the fixed hold-out test set, then apply the paired Wilcoxon signed-rank test two-sided across the N = 6 fold pairs. Declare the MSMCE improvement statistically significant only when p < 0.05; report the p-value alongside the accuracy delta (e.g. MSMCE-ResNet50 on Canine Sarcoma: 0.9043 vs 0.7235, p = 0.03). The pairing is essential: comparing independent training runs would violate the matched-folds design the paper relies on.

## Related tools

- **scikit-learn** (Provides cross-validation and metric computation (Accuracy, macro F1) that produce the paired fold-level scores fed into the Wilcoxon test)
- **PyTorch** (Trains the paired MSMCE-integrated and baseline models (ResNet50/DenseNet121) over identical 6-fold splits so fold-level outputs are matched) — https://github.com/WoodFY/MSMCE
- **Wilcoxon signed-rank test** (Two-sided paired significance test applied to the N = 6 fold pairs to compare MSMCE-integrated models against their baselines at p < 0.05)

## Examples

```
from scipy.stats import wilcoxon; stat, p = wilcoxon(acc_msmsce_folds, acc_baseline_folds, alternative='two-sided')
```

## Evaluation signals

- The comparison must use exactly N = 6 paired fold-level observations from matched cross-validation splits, matching the paper's protocol
- A significant result must yield p < 0.05 two-sided (e.g. p = 0.03 for MSMCE-ResNet50 on Canine Sarcoma accuracy 0.9043 vs 0.7235)
- Significance annotations should only appear for model-vs-its-own-baseline pairs, not across different architectures
- Reported deltas must match the paired fold data (e.g. MSMCE-DenseNet121 improved accuracy by 6.66% on CRLM, p = 0.03)
- Both Accuracy and macro-averaged F1-Score comparisons should be tested separately and each reported with its own p-value

## Limitations

- With N = 6 folds, the Wilcoxon test has limited statistical power; the smallest achievable two-sided p-value may be near 0.03
- Significance applies only to the specific model-baseline pairing on a given dataset; improvements were not evaluated for transfer learning scenarios or across datasets
- Results may not generalize to severely imbalanced class distributions or very small sample sizes, where deep models may fail to learn embedding representations

## Evidence

- [results] Wilcoxon protocol on results: "we employed the paired Wilcoxon signed-rank test to compare models integrated with MSMCE against their respective baselines"
- [results] Significance threshold and fold count: "MSMCE-ResNet50 achieved an accuracy of 0.9785, a statistically significant increase of 1.28% (p = 0.03, N = 6)"
- [results] Hold-out test evaluation basis: "their performance was subsequently evaluated on the independent hold-out test set"
- [results] CRLM significance example: "on the CRLM dataset, the accuracy of MSMCE-DenseNet121 improved by 6.66%"
- [readme] Fold protocol from README: "--k_folds 6 --in_channels 1 --spectrum_dim 12000 --embedding_channels 256 --embedding_dim 1024"
