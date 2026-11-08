---
name: multi-assay-generalizability-evaluation
description: Use when a trained prediction model must be shown to generalize across
  several related biological endpoints, by retraining the same architecture with only
  the endpoint labels changed.
when_to_use_negative:
- The goal is to compare different model architectures or image resolutions across
  assays rather than transfer the identical pipeline to new endpoint labels
- The bioassay labels are continuous and cannot be binarized into low/high groups
  using fixed normalized thresholds (e.g., < 0.4 and > 0.6)
- Only a single bioassay endpoint is available, so cross-assay generalizability cannot
  be assessed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MS2toImg
  role: Converts raw LC-MS/MS fragmentation data into single-channel 2D grayscale
    molecular fingerprint images used as CNN input for each assay
  repo: https://github.com/jsehhs/MS2toImg-CNN
- name: MS-DIAL
  role: Preprocesses raw LC-MS/MS data to extract precursor (MS1) and fragment (MS2)
    m/z pairs with intensities before image rendering
- name: TensorFlow / Keras 2.14
  role: Implements the MS2toImg-CNN architecture (two conv layers with 48 3x3 filters,
    five dense layers with dropout, sigmoid output) and image resizing to 75x70, 150x140,
    or 300x280
  repo: https://github.com/jsehhs/MS2toImg-CNN
- name: scikit-learn
  role: Provides StratifiedKFold fivefold cross-validation splits for each bioassay
    endpoint
- name: matplotlib
  role: Renders the 2D grayscale scatter-plot molecular fingerprint images at 600
    dpi
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:2f45ae0f58eb71009210189358c0ace5a3a308f2a115bcb942fe25f8056b6e70
  source_papers:
  - doi: 10.48550/arxiv.2510.09716
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/jsehhs/MS2toImg-CNN
  tools:
  - Keras
  - MS-DIAL
  - MS2toImg
  - Python
  - TensorFlow
  - TensorFlow / Keras 2.14
  - matplotlib
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# Multi-assay generalizability evaluation

## Summary

Re-applies an identical data-conversion and CNN pipeline to multiple independent bioactivity endpoints (DPPH, elastase inhibition, TFC, TPC) to demonstrate that the model's predictive performance generalizes beyond a single assay.

## When to use

Use when a trained prediction model must be shown to generalize across several related biological endpoints, by retraining the same architecture with only the endpoint labels changed.

## When NOT to use

- The goal is to compare different model architectures or image resolutions across assays rather than transfer the identical pipeline to new endpoint labels
- The bioassay labels are continuous and cannot be binarized into low/high groups using fixed normalized thresholds (e.g., < 0.4 and > 0.6)
- Only a single bioassay endpoint is available, so cross-assay generalizability cannot be assessed

## Inputs

- MS-DIAL-preprocessed MS1/MS2 m/z and intensity data for the 545 wild soybean (Glycine soja) accessions
- Binarized bioassay labels for DPPH, elastase inhibition, total flavonoid content (TFC), and total phenolic content (TPC), normalized to a 0-1 range
- MS2toImg-CNN training/evaluation code (main.py) supporting 75x70, 150x140, and 300x280 image sizes

## Outputs

- Per-assay independent test-set accuracies with standard deviations (Table 2-style summary, e.g., table2_assay_accuracies.csv)
- Fivefold cross-validation confusion matrices per assay showing diagonal values, standard deviations, and false positive/negative counts
- DPPH AUC (reference: 0.96) and comparable metrics for elastase, TFC, and TPC

## How to apply

First establish a reference result on one endpoint (e.g., MS2toImg-CNN on DPPH radical scavenging, AUC 0.96 with 5-fold StratifiedKFold cross-validation on 545 wild soybean LC-MS/MS samples rendered as 150x140 grayscale fingerprint images). Then re-run the identical pipeline — MS-DIAL preprocessing, grayscale scatter-plot image rendering (precursor m/z on x-axis, product m/z on y-axis, point size = precursor intensity, grayscale = product intensity, non-informative region above the diagonal masked to zero), binarized labels (normalized DPPH values < 0.4 = low, > 0.6 = high) — while changing only the biological endpoint labels to elastase inhibition, TFC, and TPC. Train with the same architecture (two convolutional layers with 48 3x3 filters plus max pooling, five dense layers each followed by dropout, sigmoid output, binary cross-entropy, Adam) using fivefold StratifiedKFold, evaluate on each independent test set, and record the average accuracy and standard deviation per assay. Compile the per-assay accuracies into a Table 2-style summary (e.g., table2_assay_accuracies.csv) and compare them against the reference DPPH performance.

## Related tools

- **MS2toImg** (Converts raw LC-MS/MS fragmentation data into single-channel 2D grayscale molecular fingerprint images used as CNN input for each assay) — https://github.com/jsehhs/MS2toImg-CNN
- **MS-DIAL** (Preprocesses raw LC-MS/MS data to extract precursor (MS1) and fragment (MS2) m/z pairs with intensities before image rendering)
- **TensorFlow / Keras 2.14** (Implements the MS2toImg-CNN architecture (two conv layers with 48 3x3 filters, five dense layers with dropout, sigmoid output) and image resizing to 75x70, 150x140, or 300x280) — https://github.com/jsehhs/MS2toImg-CNN
- **scikit-learn** (Provides StratifiedKFold fivefold cross-validation splits for each bioassay endpoint)
- **matplotlib** (Renders the 2D grayscale scatter-plot molecular fingerprint images at 600 dpi)

## Examples

```
python main.py 150 140
```

## Evaluation signals

- DPPH reference model achieves AUC ≈ 0.96 with 5-fold cross-validation confusion matrices showing high diagonal values, low standard deviations, and minimal false positives/negatives
- Independent test-set accuracies for elastase inhibition, TFC, and TPC are high and comparable to the DPPH performance, consistent with the reported generalizability result
- Per-assay standard deviations across the five folds are low, indicating stable performance under label changes only
- The compiled four-assay accuracy table (table2_assay_accuracies.csv) follows the Table 2 layout with mean accuracy and standard deviation per assay
- MS2toImg-CNN accuracy exceeds conventional baselines (Random Forest, PCA) across the diverse bioactivity tasks

## Limitations

- Table 2 numeric performance values are not provided in the article text, so exact expected accuracies per assay must be reproduced rather than checked against published numbers
- The repository license is NOASSERTION and the code is under peer-review embargo — redistribution, modification, or use for other purposes is not permitted until formal publication
- The model is validated only on wild soybean (Glycine soja Sieb. & Zucc.) extract data with these four bioassays; generalization to other organisms or assay types is untested
- Redistribution or use of the code for other research or commercial purposes requires contacting the authors until formal publication

## Evidence

- [results] The MS2toImg-CNN model achieved an AUC of 0.96 for discriminating high vs. low DPPH antioxidant activity samples: [evidence span withheld — anchor status: not_found]
- [methods] Re-apply identical pipeline to other assays: [evidence span withheld — anchor status: not_found]
