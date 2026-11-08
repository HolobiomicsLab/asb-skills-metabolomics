---
name: model-efficiency-tradeoff-analysis
description: Use when a CNN takes fixed-size grayscale images as input and you must
  choose among several candidate image resolutions to maximize validation accuracy
  while controlling training/inference cost.
when_to_use_negative:
- Input is an aligned feature table (tabular MS features) rather than rendered grayscale
  fingerprint images — this skill ablates the resolution of MS2toImg image inputs
  specifically
- Samples cannot be assigned to high/low activity groups (no normalized bioactivity
  values relative to the 0.4/0.6 thresholds), since stratified labeling is prerequisite
  to the cross-validated comparison
- Deployment fixes a single image size with no efficiency–accuracy tradeoff to evaluate,
  making the multi-resolution sweep unnecessary
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MS2toImg
  role: Produces the 600 dpi grayscale molecular fingerprint images that are resized
    to each candidate resolution for the sweep
  repo: https://github.com/jsehhs/MS2toImg-CNN
- name: TensorFlow
  role: Resizes the rendered fingerprint images to each target width/height and trains
    the identical CNN architecture per resolution arm
- name: Keras 2.14
  role: Defines the fixed CNN (48-filter 3×3 conv layers, five dropout FC layers,
    Adam, sigmoid, binary cross-entropy) reused unchanged across arms
- name: scikit-learn
  role: Provides StratifiedKFold fivefold cross-validation and the accuracy/recall/precision/F1
    metrics averaged per resolution
- name: pandas
  role: Writes the per-resolution metrics into the single comparison CSV table
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:991b9ed54f4f277efc2c90e88bf1e3b51bf2ea2afe5c118d2917ed907cf9a7be
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jsehhs/MS2toImg-CNN
  tools:
  - Keras
  - Keras 2.14
  - MS2toImg
  - Python 3.10
  - TensorFlow
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# model-efficiency-tradeoff-analysis

## Summary

Trains an identical CNN on grayscale 'molecular fingerprint' images rendered at multiple input resolutions (75×70, 150×140, 300×280) and compares accuracy, recall, precision, and F1 to select the resolution that balances predictive performance against computational efficiency. For the MS2toImg framework, the medium 150×140 resolution hit peak validation accuracy while 300×280 overfit and 75×70 underperformed.

## When to use

Use when a CNN takes fixed-size grayscale images as input and you must choose among several candidate image resolutions to maximize validation accuracy while controlling training/inference cost.

## When NOT to use

- Input is an aligned feature table (tabular MS features) rather than rendered grayscale fingerprint images — this skill ablates the resolution of MS2toImg image inputs specifically
- Samples cannot be assigned to high/low activity groups (no normalized bioactivity values relative to the 0.4/0.6 thresholds), since stratified labeling is prerequisite to the cross-validated comparison
- Deployment fixes a single image size with no efficiency–accuracy tradeoff to evaluate, making the multi-resolution sweep unnecessary

## Inputs

- Grayscale single-channel molecular fingerprint images (MS1 m/z vs MS2 m/z scatter plots) rendered at 600 dpi and resized to each candidate resolution (75×70, 150×140, 300×280)
- Class labels derived from normalized DPPH activity values with thresholds <0.4 (low) and >0.6 (high)
- StratifiedKFold fivefold cross-validation splits (scikit-learn)

## Outputs

- CSV table of accuracy, recall, precision, and F1 (averaged across folds) for each input resolution, ordered from smallest to largest input size
- Identified optimal resolution (150×140 for the MS2toImg DPPH task) balancing validation accuracy against computational efficiency
- Training–validation gap comparison revealing overfitting at 300×280 and underperformance at 75×70

## How to apply

Render the input images once at a high base resolution (the MS2toImg molecular fingerprint scatter images are rendered at 600 dpi) and derive each candidate size by resizing with the TensorFlow resize function, training via 'python main.py <width> <height>' for 75×70, 150×140, and 300×280. Keep every other factor fixed across arms — the same CNN (two 3×3 convolutional layers with 48 filters plus max pooling, five fully connected layers with dropout, ReLU activations, Adam optimizer, sigmoid output, binary cross-entropy loss), the same stratified labels, and the same StratifiedKFold fivefold cross-validation — so only input resolution varies. Assign class labels with fixed normalized-value thresholds (below 0.4 = low-activity group, above 0.6 = high-activity group) before splitting. After training, average accuracy, recall, precision, and F1 across folds per resolution and inspect the training–validation gap: in the MS2toImg DPPH task the highest resolution showed a large gap (overfitting), the smallest underperformed, and 150×140 achieved peak validation accuracy. Tabulate all per-resolution metrics in a single CSV ordered smallest-to-largest input size, and check that CNN performance stays at or above the conventional Random Forest baseline (RF AUC 0.817 vs CNN AUC 0.96) so accuracy loss at the cheapest resolution can be weighed against its lower computational cost.

## Related tools

- **MS2toImg** (Produces the 600 dpi grayscale molecular fingerprint images that are resized to each candidate resolution for the sweep) — https://github.com/jsehhs/MS2toImg-CNN
- **TensorFlow** (Resizes the rendered fingerprint images to each target width/height and trains the identical CNN architecture per resolution arm)
- **Keras 2.14** (Defines the fixed CNN (48-filter 3×3 conv layers, five dropout FC layers, Adam, sigmoid, binary cross-entropy) reused unchanged across arms)
- **scikit-learn** (Provides StratifiedKFold fivefold cross-validation and the accuracy/recall/precision/F1 metrics averaged per resolution)
- **pandas** (Writes the per-resolution metrics into the single comparison CSV table)

## Examples

```
python main.py 150 140
```

## Evaluation signals

- The output CSV contains all four metrics (accuracy, recall, precision, F1) for all three resolutions (75×70, 150×140, 300×280), ordered smallest to largest input size
- The CNN architecture, optimizer, and cross-validation splits are identical across arms; only the width/height arguments passed to main.py differ
- Validation accuracy of the medium-resolution arm (150×140) is the peak among the three resolutions
- The high-resolution arm (300×280) exhibits a larger training–validation gap than the medium arm (overfitting signature)
- CNN results are at or above the conventional RF baseline level (RF AUC ≈ 0.817 vs reported CNN AUC ≈ 0.96)

## Limitations

- The optimal resolution (150×140) was established only on the wild soybean (Glycine soja Sieb. & Zucc.) DPPH assay with 545 accessions; other bioassays or datasets may shift the optimum
- Only three discrete resolutions were tested; the true efficiency–accuracy optimum may lie between them, and the 600 dpi base rendering constrains achievable fidelity
- Version discrepancies exist between sources (article: Python 3.10 / Keras 2.14; README: tested with Python 3.11), so exact reproduction may vary
- The repository is under review with a NOASSERTION license — redistribution, modification, or use for other purposes is not permitted until formal publication

## Evidence

- [results] Resolution tradeoff finding: [evidence span withheld — anchor status: not_found]
