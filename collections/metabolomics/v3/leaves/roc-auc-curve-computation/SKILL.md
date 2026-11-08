---
name: roc-auc-curve-computation
description: Use when evaluating a binary bioactivity classifier (e.g., DPPH high
  vs low activity) that outputs class probabilities, and you need ROC-AUC plus accuracy/precision/recall/F1
  to compare CNN vs Random Forest baselines.
when_to_use_negative:
- Input is already a structured feature table (e.g. the merged MS-DIAL feature table
  with m/z and intensity columns) — use Random Forest or PCA/OPLS-DA baselines instead
  of the image-based CNN pipeline.
- The task is regression or multi-class prediction rather than binary classification
  of bioactivity high/low groups.
- Samples fall in the ambiguous DPPH zone (0.4 ≤ normalized value ≤ 0.6) and were
  excluded from class labeling — the ROC-AUC evaluation assumes clean binary labels.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3474
tools:
- name: TensorFlow
  role: Runs the MS2toImg-CNN forward pass to obtain class probabilities for ROC computation
  repo: https://github.com/jsehhs/MS2toImg-CNN
- name: scikit-learn
  role: Provides StratifiedKFold splitting and ROC-AUC / confusion-matrix computation
    from the CNN's probability outputs
provenance:
  source_task_ids:
  - task_003
  source_content_hash: sha256:e4e1ad98ef3f11c3532a8b75644d3ca86b1c4d583423f96ec99281a3fdca9b12
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jsehhs/MS2toImg-CNN
  tools:
  - TensorFlow
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# roc-auc-curve-computation

## Summary

Computes ROC curves and the area under the curve (AUC) from class-probability outputs of the MS2toImg-CNN model, to quantify bioactivity-classification performance on the wild soybean DPPH assay and compare against RF/PCA baselines.

## When to use

Use when evaluating a binary bioactivity classifier (e.g., DPPH high vs low activity) that outputs class probabilities, and you need ROC-AUC plus accuracy/precision/recall/F1 to compare CNN vs Random Forest baselines.

## When NOT to use

- Input is already a structured feature table (e.g. the merged MS-DIAL feature table with m/z and intensity columns) — use Random Forest or PCA/OPLS-DA baselines instead of the image-based CNN pipeline.
- The task is regression or multi-class prediction rather than binary classification of bioactivity high/low groups.
- Samples fall in the ambiguous DPPH zone (0.4 ≤ normalized value ≤ 0.6) and were excluded from class labeling — the ROC-AUC evaluation assumes clean binary labels.

## Inputs

- Grayscale 2D molecular fingerprint images derived from raw LC-MS/MS data (MS1 m/z on x-axis, MS2 m/z on y-axis, non-informative region above the diagonal masked to zero)
- DPPH bioassay labels with normalized activity values (low < 0.4, high > 0.6) for 545 wild soybean accessions
- Trained MS2toImg-CNN model (TensorFlow/Keras 2.14, Python 3.10) with sigmoid output layer

## Outputs

- ROC curve plot (Figure 3B) per fold and aggregated
- ROC-AUC score (reported ~0.96 for the DPPH task)
- Confusion matrices from 5-fold stratified cross-validation
- Accuracy, precision, recall, and F1 metrics (Figure 5 box plots)
- Results table in CSV format

## How to apply

First binarize the DPPH bioassay response: normalized DPPH values < 0.4 form the low-activity group and > 0.6 the high-activity group, using scikit-learn's StratifiedKFold with k=5. Feed the grayscale molecular fingerprint images (e.g. 75x70, 150x140, or 300x280 pixels) into the MS2toImg-CNN whose sigmoid output layer yields class probabilities. For each fold, compute ROC-AUC from the probability scores and generate the ROC curve (Figure 3B), accumulating test-set predictions to derive accuracy, precision, recall, and F1 as shown in the Figure 5 box plots. Validate that the CNN's AUC (~0.96 on the DPPH task) meets or exceeds the article's reported value, which is well above the Random Forest baseline AUC of 0.817.

## Related tools

- **TensorFlow** (Runs the MS2toImg-CNN forward pass to obtain class probabilities for ROC computation) — https://github.com/jsehhs/MS2toImg-CNN
- **scikit-learn** (Provides StratifiedKFold splitting and ROC-AUC / confusion-matrix computation from the CNN's probability outputs)

## Examples

```
python main.py 75 70
```

## Evaluation signals

- Reported CNN AUC on the DPPH task is ~0.96; a substantially lower AUC indicates incorrect image encoding, split leakage, or a mis-specified sigmoid output.
- CNN AUC should exceed the Random Forest baseline AUC of 0.817 on the same DPPH task.
- 5-fold stratified confusion matrices should show balanced, stable classification with minimal false positives and false negatives.
- Accuracy, precision, recall, and F1 (Figure 5 box plots) should all be > 0.90 for the CNN on the DPPH task.
- Prediction probabilities must come from a sigmoid output layer over binary DPPH high/low labels — verify labels were assigned with the < 0.4 / > 0.6 cutoffs before scoring.

## Limitations

- The ROC-AUC evaluation inherits the image-based approach's reliance on the overall fragmentation landscape; it is less sensitive to alignment errors but cannot attribute predictions to specific molecular features.
- Performance was reported on wild soybean (Glycine soja) DPPH data; transfer to other organisms or bioassays was not quantified in the ROC-AUC metrics provided.
- Numeric values for accuracy/precision/recall/F1 beyond the AUC (~0.96) and RF baseline (0.817) are not given in the article text (they are only visible in figures), so exact numeric targets must be read from Figure 3B and Figure 5.
- Table 2 numeric performance values and Supplementary Table 1 hyperparameters are not provided in the text, limiting exact reproduction of the ROC-AUC benchmark.

## Evidence

- [methods] DPPH AUC from CNN with image input: [evidence span withheld — anchor status: not_found]
- [methods] Probability output layer: [evidence span withheld — anchor status: not_found]
- [results] ROC curve generation: [evidence span withheld — anchor status: not_found]
- [methods] Metrics validation: [evidence span withheld — anchor status: not_found]
- [results] ROC curve and metrics: [evidence span withheld — anchor status: not_found]
- [methods] Tools used: [evidence span withheld — anchor status: not_found]
