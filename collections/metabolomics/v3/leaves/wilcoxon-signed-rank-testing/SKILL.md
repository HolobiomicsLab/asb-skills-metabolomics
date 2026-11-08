---
name: wilcoxon-signed-rank-testing
description: Use when two classifiers (e.g., MSMCE-ResNet50 vs. ResNet50 baseline)
  have been evaluated on the same N = 6 stratified CV folds and you must test whether
  the paired metric improvement (Accuracy, F1) is statistically significant.
when_to_use_negative:
- Metrics come from independent (unpaired) runs rather than the same fold splits —
  the paired signed-rank assumption is violated
- The comparison involves ablation variants trained on the entire training set (not
  k-fold split) and evaluated only on the hold-out test set, so no per-fold paired
  samples exist
- Sample size or class balance is extreme (e.g., severely imbalanced class distributions
  or too-few samples), which the paper flags as limiting model performance and hence
  the reliability of the comparison
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scikit-learn
  role: Provides stratified k-fold splitting and metric computation (Accuracy, Precision,
    Recall, F1) that generate the paired per-fold values fed into the test
  repo: https://github.com/WoodFY/MSMCE
- name: pandas
  role: Assembles per-model metrics and p-values into results_table.csv / nsclc_results_table.csv
- name: PyTorch
  role: Trains the MSMCE-integrated and baseline deep models whose paired fold-wise
    outputs are compared
  repo: https://github.com/WoodFY/MSMCE
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1371/journal.pone.0321239
    title: 'MSMCE: A novel representation module for classification of raw mass spectrometry
      data'
  source_content_hash: sha256:ac8e6fd5de39acc53f745b3d4a862d34c6cecb67be8454b8e11e59bd4b27ec5b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WoodFY/MSMCE
  tools:
  - MSMCE (WoodFY/MSMCE repository)
  - PyTorch
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Wilcoxon signed-rank testing

## Summary

Applies the paired, two-sided Wilcoxon signed-rank test to determine whether performance metrics of MSMCE-integrated models differ significantly from their single-channel baselines across k-fold cross-validation splits, using p < 0.05 as the significance threshold.

## When to use

Use when two classifiers (e.g., MSMCE-ResNet50 vs. ResNet50 baseline) have been evaluated on the same N = 6 stratified CV folds and you must test whether the paired metric improvement (Accuracy, F1) is statistically significant.

## When NOT to use

- Metrics come from independent (unpaired) runs rather than the same fold splits — the paired signed-rank assumption is violated
- The comparison involves ablation variants trained on the entire training set (not k-fold split) and evaluated only on the hold-out test set, so no per-fold paired samples exist
- Sample size or class balance is extreme (e.g., severely imbalanced class distributions or too-few samples), which the paper flags as limiting model performance and hence the reliability of the comparison

## Inputs

- Paired per-fold performance metric arrays (N = 6 folds) for the MSMCE-integrated model and its single-channel baseline, from a fixed-seed stratified k-fold split
- Per-model Accuracy/Precision/Recall/F1 values (macro averaging for multi-class, binary here) evaluated on the independent 10% hold-out test set

## Outputs

- Two-sided Wilcoxon signed-rank p-value per model-vs-baseline comparison
- Significance decision at the p < 0.05 threshold
- Augmented results table (e.g., results_table.csv, nsclc_results_table.csv) with per-model metrics and p-values

## How to apply

Collect the per-fold metric values (Accuracy, Precision, Recall, or macro-averaged F1) for the MSMCE-integrated model and its baseline from the same fixed fold splits (identical random seed), then run the paired Wilcoxon signed-rank test two-sided over the N = 6 fold pairs. Declare the improvement significant only if p < 0.05; in this study the NSCLC accuracy gain of MSMCE-ResNet50 (0.9785 vs. 0.9661, +1.28%) and the RCC DenseNet121 F1 gain (0.6946 to 0.7952) each yielded p = 0.03. Record the p-value alongside per-model metrics in the results table (e.g., results_table.csv / nsclc_results_table.csv) and, per the supplementary protocol, complement the point estimate with bootstrap 95% confidence intervals computed from the k-fold cross-validation metrics on the hold-out test set.

## Related tools

- **scikit-learn** (Provides stratified k-fold splitting and metric computation (Accuracy, Precision, Recall, F1) that generate the paired per-fold values fed into the test) — https://github.com/WoodFY/MSMCE
- **pandas** (Assembles per-model metrics and p-values into results_table.csv / nsclc_results_table.csv)
- **PyTorch** (Trains the MSMCE-integrated and baseline deep models whose paired fold-wise outputs are compared) — https://github.com/WoodFY/MSMCE

## Evaluation signals

- The test is paired and two-sided over N = 6 folds, matching the paper's protocol (k_folds = 6 in the README commands)
- Significance threshold is p < 0.05; the reported landmark results give p = 0.03 for both the NSCLC accuracy gain (+1.28%, 0.9785 vs. 0.9661) and the RCC DenseNet121 F1 gain (0.6946 to 0.7952)
- Fold splits use an identical fixed random seed across MSMCE-integrated and baseline models so pairs are genuinely matched
- Output table exists with accuracy/F1 values for each model row confined to the [0, 1] range
- Point estimates should be accompanied by bootstrap 95% confidence intervals from the k-fold cross-validation metrics

## Limitations

- With only N = 6 fold pairs, the minimum achievable two-sided p-value is limited; significance depends on consistent fold-wise improvements
- Performance may be limited in extreme cases such as severely imbalanced class distributions or too-small sample sizes, undermining the reliability of the metric comparison
- Interpretability of the learned features is not established, so a significant p-value does not explain which m/z regions drive the improvement

## Evidence

- [results] Wilcoxon test protocol: "we employed the paired Wilcoxon signed-rank test to compare models integrated with MSMCE against their respective baselines"
- [results] NSCLC significant result: "MSMCE-ResNet50 achieved an accuracy of 0.9785, a statistically significant increase of 1.28% (p = 0.03, N = 6) compared to the original model accuracy of 0.9661"
- [results] RCC significant result: "on the RCC dataset, the F1-Score for the DenseNet121 increased from 0.6946 to 0.7952"
- [supplementary] Bootstrap CI complement: "Bootstrap 95% confidence intervals for mean performance metrics (Accuracy, Precision, Recall, F1-Score) from k-fold cross-validation on the hold-out test set"
- [readme] Fold count in commands: "python exp_multi_channel_embedding.py --model_name MultiChannelEmbeddingResNet50 --dataset nsclc --num_classes 2 --k_folds 6 --in_channels 1"
