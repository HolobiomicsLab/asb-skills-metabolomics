---
name: classification-metric-aggregation-across-folds
description: Use when a classification pipeline (CNN on molecular fingerprint images,
  Random Forest, PCA, OPLS-DA) has been trained under StratifiedKFold fivefold cross-validation
  and per-fold accuracy, recall, precision and F1 must be averaged into one comparable
  table.
when_to_use_negative:
- The analysis is a regression or ranking task without discrete class labels — accuracy,
  precision and F1 are not defined.
- Only a single train/test split is available; per-fold averaging would be meaningless
  and the training–validation gap cannot be estimated across folds.
- Class labels are not derived from the bioassay thresholds (e.g. DPPH normalized
  value <0.4 or >0.6), so fold stratification no longer reflects the high/low-activity
  balance.
tools:
- name: scikit-learn
  role: Provides StratifiedKFold fivefold cross-validation splitting used to generate
    the per-fold evaluation results that are averaged
- name: pandas
  role: Used to assemble the per-fold metrics into a single CSV table ordered by condition
- name: TensorFlow
  role: Trains the CNN models whose fold-level metrics are aggregated
- name: Keras
  role: Implements the CNN architecture (two 48-filter 3×3 convolutional layers, max
    pooling, five dropout-regularized fully connected layers, sigmoid output, binary
    cross-entropy) whose validation metrics are averaged
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:4f1c088c268b00ddbe3a670c4ae44a48e57394974b4e63c5ee1f4d209dbbbf00
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Keras
  - MS2toImg
  - Python 3.10
  - TensorFlow
  - pandas
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# classification-metric-aggregation-across-folds

## Summary

Aggregates accuracy, recall, precision and F1 across the fivefold StratifiedKFold cross-validation folds of a bioactivity classifier, producing a single CSV metric table per model/configuration so competing variants (e.g. different CNN image resolutions or RF baselines) can be compared on an equal footing.

## When to use

Use when a classification pipeline (CNN on molecular fingerprint images, Random Forest, PCA, OPLS-DA) has been trained under StratifiedKFold fivefold cross-validation and per-fold accuracy, recall, precision and F1 must be averaged into one comparable table.

## When NOT to use

- The analysis is a regression or ranking task without discrete class labels — accuracy, precision and F1 are not defined.
- Only a single train/test split is available; per-fold averaging would be meaningless and the training–validation gap cannot be estimated across folds.
- Class labels are not derived from the bioassay thresholds (e.g. DPPH normalized value <0.4 or >0.6), so fold stratification no longer reflects the high/low-activity balance.

## Inputs

- Per-fold evaluation records (accuracy, recall, precision, F1) from five StratifiedKFold cross-validation runs
- Trained model predictions on each held-out fold (e.g. CNN on 2D grayscale molecular fingerprint images or RF on the unified feature table)

## Outputs

- CSV table with the four metrics (accuracy, recall, precision, F1) averaged across folds, ordered from smallest to largest input size or condition
- Aggregated metric values used to compare configurations (e.g. resolution 75×70 vs 150×140 vs 300×280) against baselines

## How to apply

After training the identical model architecture under each fold of a scikit-learn StratifiedKFold fivefold split (e.g. DPPH high-activity >0.6 vs low-activity <0.4 labels), compute accuracy, recall, precision and F1 on the held-out fold for each of the five runs. Average each of the four metrics across the five folds to obtain one value per metric per configuration. Tabulate the aggregated metrics in a single CSV ordered by the experimental condition being varied (e.g. input resolution 75×70, 150×140, 300×280 from smallest to largest). Use these averages to judge trade-offs — e.g. the medium 150×140 resolution gave the peak validation accuracy while 300×280 overfit (large training–validation gap) and 75×70 underperformed — and to benchmark against baselines such as the Random Forest AUC of 0.817 versus the CNN's reported AUC of 0.96.

## Related tools

- **scikit-learn** (Provides StratifiedKFold fivefold cross-validation splitting used to generate the per-fold evaluation results that are averaged)
- **pandas** (Used to assemble the per-fold metrics into a single CSV table ordered by condition)
- **TensorFlow** (Trains the CNN models whose fold-level metrics are aggregated)
- **Keras** (Implements the CNN architecture (two 48-filter 3×3 convolutional layers, max pooling, five dropout-regularized fully connected layers, sigmoid output, binary cross-entropy) whose validation metrics are averaged)

## Examples

```
python main.py 150 140
```

## Evaluation signals

- The output table contains all four metrics (accuracy, recall, precision, F1) for every configuration evaluated (e.g. all three resolutions 75×70, 150×140, 300×280).
- Each reported value is the mean across the five StratifiedKFold folds, not a single fold's result.
- Configurations are ordered in the table from smallest to largest input size (75×70 → 150×140 → 300×280).
- Aggregated CNN metrics are at or above the conventional RF baseline (RF AUC ≈ 0.817 vs CNN AUC ≈ 0.96 reported for the framework) before any accuracy loss at low resolution is weighed against computational cost.
- The medium resolution (150×140) should show the peak validation accuracy with a smaller training–validation gap than 300×280, consistent with the reported overfitting/underperformance pattern.

## Limitations

- Numeric per-resolution metric values (e.g. Table 2 contents) are not published in the manuscript text, so the aggregated numbers cannot be independently verified without re-running the pipeline.
- The comparison is only meaningful when the identical architecture and hyperparameters are reused across folds and configurations; changing hyperparameters between folds invalidates the averages.
- License in the repository is marked NOASSERTION and redistribution/modification/use is not permitted until formal publication, so re-running the pipeline to reproduce the metric tables may require author permission.

## Evidence

- [other] Evaluate each trained model on the four standard metrics — accuracy, recall, precision, and F1 — averaged across the cross-validation folds.: [evidence span withheld — anchor status: not_found]
- [other] Tabulate the per-resolution metrics in a single CSV table ordered from smallest to largest input size.: [evidence span withheld — anchor status: not_found]
- [other] medium resolution (150×140) provided the optimal accuracy-vs-efficiency tradeoff: [evidence span withheld — anchor status: not_found]
- [methods] Samples with normalized values below 0.4 were categorized as the low-activity group, while those above 0.6 were categorized as the high-activity group.: "Samples with normalized values below 0.4 were categorized as the low-activity group, while those above 0.6 were categorized as the high-activity group."
- [other] split samples with StratifiedKFold fivefold cross-validation from scikit-learn: [evidence span withheld — anchor status: not_found]
