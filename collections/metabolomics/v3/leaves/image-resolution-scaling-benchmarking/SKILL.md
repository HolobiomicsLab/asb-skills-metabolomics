---
name: image-resolution-scaling-benchmarking
description: Use when a pipeline encodes spectra into fixed-size images and you must
  decide which input resolution balances validation accuracy against training cost
  before committing to a production model.
when_to_use_negative:
- The input is already a feature table or spectral peak list — this skill applies
  only to image-encoded model inputs, not to tabular ML baselines like Random Forest.
- You have a single pre-chosen image size and no need to compare resolutions — running
  the full resolution sweep adds compute without decision value.
- The downstream task is not image-based deep learning (e.g. PCA or OPLS-DA on feature
  matrices), where resolution scaling is not a tunable parameter.
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MS2toImg
  role: Converts raw LC-MS/MS spectra into the grayscale molecular fingerprint images
    that are subsequently resized to each benchmark resolution
  repo: https://github.com/jsehhs/MS2toImg-CNN
- name: TensorFlow
  role: Provides the resize function used to produce each target image resolution
    and the CNN training backend for the benchmark runs
  repo: https://github.com/jsehhs/MS2toImg-CNN
- name: Keras 2.14
  role: High-level API used to define and train the fixed CNN architecture (48-filter
    3×3 conv layers, dropout, sigmoid output) at each resolution
  repo: https://github.com/jsehhs/MS2toImg-CNN
- name: scikit-learn
  role: Supplies StratifiedKFold fivefold cross-validation used to split samples identically
    for every resolution run
  repo: https://github.com/jsehhs/MS2toImg-CNN
provenance:
  source_task_ids:
  - task_006
  source_content_hash: sha256:a0c508b0377b0f6b8fa66fdf4237291eaf2aa63a083b34136d62a4f1e621c5f7
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# Image-resolution-scaling benchmarking

## Summary

Benchmarks CNN predictive performance on molecular fingerprint images rendered at multiple input resolutions (e.g. 75×70, 150×140, 300×280 px) to find the accuracy-versus-computational-efficiency sweet spot for a fixed architecture.

## When to use

Use when a pipeline encodes spectra into fixed-size images and you must decide which input resolution balances validation accuracy against training cost before committing to a production model.

## When NOT to use

- The input is already a feature table or spectral peak list — this skill applies only to image-encoded model inputs, not to tabular ML baselines like Random Forest.
- You have a single pre-chosen image size and no need to compare resolutions — running the full resolution sweep adds compute without decision value.
- The downstream task is not image-based deep learning (e.g. PCA or OPLS-DA on feature matrices), where resolution scaling is not a tunable parameter.

## Inputs

- Pre-rendered grayscale molecular fingerprint image sets at multiple resolutions (75×70, 150×140, 300×280 px), produced by MS2toImg from LC-MS/MS data
- Bioactivity class labels per sample (DPPH normalized-value groups: <0.4 low, >0.6 high)
- A fixed CNN architecture definition (Keras/TensorFlow) held constant across resolutions

## Outputs

- CSV table of accuracy, recall, precision and F1 (fivefold cross-validated) for each input resolution, ordered smallest to largest
- Identified optimal resolution (e.g. 150×140) that maximizes validation accuracy while limiting overfitting and compute cost

## How to apply

Render the same source data — here grayscale 'molecular fingerprint' scatter plots with MS1 m/z on the x-axis and MS2 m/z on the y-axis, generated at 600 dpi and resized with the TensorFlow resize function — to several target resolutions, invoking e.g. 'python main.py <width> <height>'. Keep everything else fixed across runs: identical DPPH class labels (normalized values below 0.4 = low-activity, above 0.6 = high-activity), StratifiedKFold fivefold cross-validation, and the same Keras/TensorFlow CNN (two 48-filter 3×3 convolutional layers with max pooling, five fully connected layers with dropout, ReLU activations, Adam optimizer, sigmoid output, binary cross-entropy loss). For each resolution compute accuracy, recall, precision and F1 averaged across folds, and tabulate the metrics in a single CSV ordered from smallest to largest input size. Interpret the table by looking for a training–validation gap: in the MS2toImg study the 300×280 high-resolution model overfit, the 75×70 low-resolution model underperformed, and the medium 150×140 resolution achieved the peak validation accuracy, so it was chosen as the optimal tradeoff. Validate the exercise by checking that CNN metrics stay at or above the conventional RF baseline (e.g. RF AUC 0.817 vs CNN AUC 0.96 reported for the framework) before trading accuracy for lower computational cost.

## Related tools

- **MS2toImg** (Converts raw LC-MS/MS spectra into the grayscale molecular fingerprint images that are subsequently resized to each benchmark resolution) — https://github.com/jsehhs/MS2toImg-CNN
- **TensorFlow** (Provides the resize function used to produce each target image resolution and the CNN training backend for the benchmark runs) — https://github.com/jsehhs/MS2toImg-CNN
- **Keras 2.14** (High-level API used to define and train the fixed CNN architecture (48-filter 3×3 conv layers, dropout, sigmoid output) at each resolution) — https://github.com/jsehhs/MS2toImg-CNN
- **scikit-learn** (Supplies StratifiedKFold fivefold cross-validation used to split samples identically for every resolution run) — https://github.com/jsehhs/MS2toImg-CNN

## Examples

```
python main.py 150 140
```

## Evaluation signals

- The output CSV contains all four metrics (accuracy, recall, precision, F1) for every tested resolution and is ordered from smallest to largest input size.
- Each resolution was trained with the identical architecture, optimizer, and fivefold StratifiedKFold splits, so metric differences are attributable to resolution alone.
- A training–validation gap is visible at the largest resolution (300×280 overfit) while the medium resolution attains peak validation accuracy.
- The selected resolution's CNN metrics remain at or above the conventional RF baseline level (e.g. RF AUC 0.817 vs CNN AUC 0.96).

## Limitations

- High-resolution inputs can overfit (large training–validation gap) even though they carry more pixel detail; low-resolution inputs lose discriminative signal and underperform.
- The benchmark holds the architecture, labels, and fold splits constant, so conclusions about resolution apply only to that fixed model/data combination.
- The article does not report wall-clock or memory measurements per resolution, so computational-efficiency claims are qualitative rather than quantitative.

## Evidence

- [supplementary] images rendered at 600 dpi and resized with TensorFlow resize: [evidence span withheld — anchor status: not_found]
