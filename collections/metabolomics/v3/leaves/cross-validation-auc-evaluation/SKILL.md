---
name: cross-validation-auc-evaluation
description: Use when a trained classifier must be validated with 10-fold cross-validation
  and micro-average AUC, followed by Youden-index threshold optimization and per-metric
  reporting (accuracy, TPR, FPR, precision, F1).
when_to_use_negative:
- Inputs are already per-fold AUC values — averaging fold AUCs directly is NOT equivalent
  to the micro-average AUC over pooled validation predictions used here
- The task is multi-class or ranking evaluation rather than binary pathway presence/absence
  classification with a tunable threshold
- No labeled positive/negative pathway sets exist (e.g., purely unsupervised network
  exploration)
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: M5
  role: Very deep 1D convolutional network (convolution/ReLU/batch-norm/max-pool blocks,
    stride 8) whose held-out fold predictions feed the micro-average AUC computation
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: result_optimization_and_analysis
  role: Repository module that optimizes the classification threshold with Youden's
    J statistic and computes AUC, accuracy, recall, precision, F1, and FPR
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: PyTorch
  role: Backend in which the M5 model, training loop (Adam optimizer), and cross-validation
    are implemented
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_003
  source_content_hash: sha256:a37f746f93739b326f4f5088f1b810a84a148f89f7d8114e610d003d31b69b30
  source_papers:
  - doi: 10.48550/arxiv.2410.22030
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
  tools:
  - Adam optimizer
  - M5
  - NetAurHPD
  - PyTorch
  - Rstudio
  - pcaMethods
  - result_optimization_and_analysis
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Cross-validation AUC evaluation

## Summary

Evaluates a binary pathway-prediction model (M5 trained on auralized pathway waveform signals) with 10-fold cross-validation, pooling all validation-fold predictions into a micro-average AUC, then selecting an optimal classification threshold via the Youden index and reporting accuracy, recall, FPR, precision, and F1.

## When to use

Use when a trained classifier must be validated with 10-fold cross-validation and micro-average AUC, followed by Youden-index threshold optimization and per-metric reporting (accuracy, TPR, FPR, precision, F1).

## When NOT to use

- Inputs are already per-fold AUC values — averaging fold AUCs directly is NOT equivalent to the micro-average AUC over pooled validation predictions used here
- The task is multi-class or ranking evaluation rather than binary pathway presence/absence classification with a tunable threshold
- No labeled positive/negative pathway sets exist (e.g., purely unsupervised network exploration)

## Inputs

- Pooled predicted scores/labels from all validation folds of a binary pathway classifier (e.g., M5 predictions on auralized pathway signals)
- Ground-truth positive/negative pathway labels (e.g., 169 TomatoCyc positive pathways vs. MetaCyc and 85 randomly sampled negative pathways)
- Model configuration parameters (learning rate, epochs, stride) from config.py

## Outputs

- Micro-average AUC over all validation-fold predictions
- Optimal classification threshold (Youden index, argmax of TPR - FPR)
- Per-year metrics table: accuracy, TPR/recall, FPR, precision, F1 (e.g., results_table.tsv)
- AUC distribution summaries (median, mean, max, IQR, whisker range) per augmentation setting

## How to apply

Run 10-fold cross-validation where each labeled pathway appears exactly once in a validation set; for the NetAurHPD ablation this means 20 sessions per year with balanced positive-negative sets of 126 TomatoCyc pathways each. Collect predictions from all validation folds and compute a single micro-average AUC over the pooled scores rather than averaging per-fold AUCs. Apply the Youden index — the threshold maximizing TPR - FPR over the ROC curve — to convert pooled scores into class predictions. Report accuracy, TPR/recall, FPR, precision, and F1 at the optimized threshold, and compare the micro-average AUC against reported baselines (NetAurHPD: 0.87 in 2001, 0.857 in 2003, 0.884 in 2004; accuracy around 0.8). For augmentation ablations, repeat the procedure per number of Augmentations (1/3/5/7) across sessions and aggregate AUC distributions into box plots with median, mean, maximum, interquartile range, and whisker range.

## Related tools

- **M5** (Very deep 1D convolutional network (convolution/ReLU/batch-norm/max-pool blocks, stride 8) whose held-out fold predictions feed the micro-average AUC computation) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **result_optimization_and_analysis** (Repository module that optimizes the classification threshold with Youden's J statistic and computes AUC, accuracy, recall, precision, F1, and FPR) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **PyTorch** (Backend in which the M5 model, training loop (Adam optimizer), and cross-validation are implemented) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method

## Examples

```
optimize_results_and_calc_metrics(y_test, y_pred)
```

## Evaluation signals

- Micro-average AUC on held-out folds should approximate 0.87 (2001), 0.857 (2003), and 0.884 (2004) on the tomato pericarp Metabolite CNs
- Accuracy after threshold optimization should be around 0.8
- Each of the labeled pathways must appear exactly once in a validation set across the 10 folds
- Optimized threshold equals the argmax of TPR - FPR over candidate thresholds
- Reported NetAurHPD rows in Table 1(a)-(c) (accuracy, recall, FPR, precision, F1) should be reproduced when assembling results_table.tsv

## Limitations

- Models do not consistently improve with more data; in some sessions the AUCs of all models decreased, so cross-session variability must be reported
- NetAurHPD has a low level of explainability and cannot provide biologists with a full understanding of its results
- README defaults (lr=0.01, epochs=40) differ from the paper's training settings (lr 0.0001/0.00001, 300-800 epochs); configuration must be set explicitly

## Evidence

- [methods] Youden threshold method: [evidence span withheld — anchor status: not_found]
- [methods] Expected AUC values: [evidence span withheld — anchor status: not_found]
