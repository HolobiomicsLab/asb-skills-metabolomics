---
name: youden-index-threshold-optimization
description: Use when a binary classifier (e.g., M5 sigmoid outputs on averaged pathway
  waveform signals) yields probability scores that must be converted into class labels
  and no operating point is fixed — pick the cutoff maximizing TPR − FPR.
when_to_use_negative:
- When an operating threshold is already fixed by domain requirements (e.g., a predefined
  detection cutoff) — Youden optimization would override it
- When only a threshold-free ranking metric is needed (e.g., reporting micro-average
  AUC alone); thresholding adds no value and may overfit to the validation folds
- When predictions are not continuous scores (already hard labels), since no threshold
  sweep is possible
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: NetAurHPD
  role: Framework whose result_optimization_and_analysis module implements threshold
    optimization via Youden's J Statistic and computes AUC, accuracy, recall, precision,
    F1, and FPR
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- name: M5
  role: Convolutional classifier producing the sigmoid probability scores that the
    Youden-index threshold is optimized on
  repo: https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_content_hash: sha256:0945dc0d615967e5bdafd95ce60c9c884ba8ae2b60f00ba4a09924d4e6695704
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
  - Rstudio
  - pcaMethods
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# youden-index-threshold-optimization

## Summary

Selects the optimal binary classification cutoff by maximizing the Youden index (TPR − FPR) over threshold sweeps, as used in NetAurHPD to convert M5 sigmoid scores on auralized pathway waveforms into pathway presence/absence predictions before computing accuracy, TPR, FPR, precision, and F1.

## When to use

Use when a binary classifier (e.g., M5 sigmoid outputs on averaged pathway waveform signals) yields probability scores that must be converted into class labels and no operating point is fixed — pick the cutoff maximizing TPR − FPR.

## When NOT to use

- When an operating threshold is already fixed by domain requirements (e.g., a predefined detection cutoff) — Youden optimization would override it
- When only a threshold-free ranking metric is needed (e.g., reporting micro-average AUC alone); thresholding adds no value and may overfit to the validation folds
- When predictions are not continuous scores (already hard labels), since no threshold sweep is possible

## Inputs

- Predicted probability/score tensor (y_pred) from the M5 classifier over all validation-fold pathways
- Ground-truth label tensor (y_test) for the same pathways (positive: TomatoCyc pathways; negative: 85 randomly sampled negative pathways)
- Trained M5 classifier outputs from 10-fold cross-validation

## Outputs

- Optimal classification threshold (Youden index argmax of TPR − FPR)
- Binarized pathway presence/absence predictions
- Metrics table: accuracy, recall (TPR), FPR, precision, F1, and AUC

## How to apply

After training M5 as a binary classifier (BCE loss, Adam optimizer, learning rate 0.0001, stride 8) and collecting all validation-fold predictions from 10-fold cross-validation, sweep decision thresholds over the predicted scores and compute TPR and FPR at each. Choose the threshold that maximizes the Youden index (TPR − FPR), i.e., the ROC point farthest above the diagonal. Apply that optimized threshold to binarize predictions and then report accuracy, TPR (recall), FPR, precision, and F1 alongside the threshold-independent micro-average AUC. In the NetAurHPD codebase this is implemented in the result_optimization_and_analysis module, which both optimizes the threshold via Youden's J Statistic and calculates the evaluation metrics. The rationale is that micro-average AUC alone does not give an operational decision rule for flagging predicted metabolic pathways (e.g., TomatoCyc positives vs. 85 randomly sampled negative pathways over 52 common metabolites).

## Related tools

- **NetAurHPD** (Framework whose result_optimization_and_analysis module implements threshold optimization via Youden's J Statistic and computes AUC, accuracy, recall, precision, F1, and FPR) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method
- **M5** (Convolutional classifier producing the sigmoid probability scores that the Youden-index threshold is optimized on) — https://github.com/TamirBar-Tov/NetAurHPD-Network-Auralization-Hyperlink-Prediction-Method

## Examples

```
optimize_results_and_calc_metrics(y_test, y_pred)
```

## Evaluation signals

- The selected threshold must equal the argmax of TPR − FPR over the swept ROC thresholds
- Binarized predictions at the optimized threshold should yield accuracy around 0.8 on the tomato pericarp datasets (as reported for 2001/2003/2004)
- Micro-average AUC computed from raw (pre-threshold) validation-fold predictions should be approximately 0.87 (2001), 0.857 (2003), and 0.884 (2004)
- Reported metric set must include accuracy, TPR/recall, FPR, precision, and F1 computed at the optimized cutoff, matching the NetAurHPD rows of Table 1(a)-(c)

## Limitations

- The Youden-optimal threshold is learned from the same cross-validation predictions it is evaluated on, so metrics at that cutoff may be optimistic relative to fully held-out data
- NetAurHPD (including the thresholded classifier) has a low level of explainability and cannot provide biologists with a full understanding of its results
- Models do not consistently improve with more data; in some sessions the AUCs of all models decreased, so the optimized cutoff should be re-derived per session/year rather than reused

## Evidence

- [methods] Youden index definition and purpose: [evidence span withheld — anchor status: not_found]
