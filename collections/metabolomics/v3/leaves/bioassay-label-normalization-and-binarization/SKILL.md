---
name: bioassay-label-normalization-and-binarization
description: Use when raw bioassay measurements (continuous potency values from wild
  soybean or similar natural-product extracts) must become binary class labels for
  a supervised classifier such as the MS2toImg-CNN.
when_to_use_negative:
- The bioassay data are already categorical or thresholded class labels — re-normalizing
  and re-binarizing would discard information or introduce arbitrary cutoffs
- The endpoint of interest has no natural 0–1 range (e.g. an unbounded continuous
  physicochemical property) and no article-defined low/high cutoff is specified
- The task is regression or ranking of bioactivity, not binary classification of high-
  vs. low-activity groups
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: scikit-learn
  role: Used to perform the min–max normalization of bioassay values and to supply
    the StratifiedKFold splitter that consumes the resulting binary labels during
    fivefold CNN cross-validation
- name: MS2toImg-CNN
  role: Consumes the binarized labels as targets for training the convolutional model
    on grayscale molecular fingerprint images
  repo: https://github.com/jsehhs/MS2toImg-CNN
provenance:
  source_task_ids:
  - task_007
  source_content_hash: sha256:80dd6ff0db017c2d051da3017609c456798209c8c2827ebda4e749389a3fd6ba
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
  - MS2toImg-CNN
  - Python
  - TensorFlow
  - matplotlib
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# bioassay-label-normalization-and-binarization

## Summary

Normalizes measured bioassay endpoints (e.g. DPPH radical scavenging, elastase inhibition, TFC, TPC) to a 0–1 range and binarizes them into low- versus high-activity classes using article-specified cutoffs so they can serve as CNN training labels.

## When to use

Use when raw bioassay measurements (continuous potency values from wild soybean or similar natural-product extracts) must become binary class labels for a supervised classifier such as the MS2toImg-CNN.

## When NOT to use

- The bioassay data are already categorical or thresholded class labels — re-normalizing and re-binarizing would discard information or introduce arbitrary cutoffs
- The endpoint of interest has no natural 0–1 range (e.g. an unbounded continuous physicochemical property) and no article-defined low/high cutoff is specified
- The task is regression or ranking of bioactivity, not binary classification of high- vs. low-activity groups

## Inputs

- Bioassay measurement table for 545 wild soybean (Glycine soja Sieb. & Zucc.) accessions with per-sample DPPH, elastase inhibition, TFC, and/or TPC endpoint values

## Outputs

- Per-sample binary activity labels (low vs. high) for each bioassay endpoint, ready as CNN training/validation targets

## How to apply

First, min–max normalize each bioassay measurement so the endpoint values span the 0–1 range; this places all four endpoints (DPPH, elastase inhibition, TFC, TPC) on a comparable scale. Then binarize the normalized values using the article's cutoff: samples with normalized values below 0.4 are assigned to the low-activity group and those above 0.6 to the high-activity group, with the intermediate band left as unlabeled. Apply the same normalization and binarization independently to each biological endpoint, changing only the labels used for training while keeping the LC-MS/MS-derived image inputs identical. The rationale, from the MS2toImg-CNN workflow, is that raw assay readouts are not directly usable as class targets and that a consistent 0.4/0.6 cutoff rule generalizes across the DPPH, elastase, TFC, and TPC tasks.

## Related tools

- **scikit-learn** (Used to perform the min–max normalization of bioassay values and to supply the StratifiedKFold splitter that consumes the resulting binary labels during fivefold CNN cross-validation)
- **MS2toImg-CNN** (Consumes the binarized labels as targets for training the convolutional model on grayscale molecular fingerprint images) — https://github.com/jsehhs/MS2toImg-CNN

## Examples

```
from sklearn.preprocessing import MinMaxScaler; y_norm = MinMaxScaler().fit_transform(dpph_raw.reshape(-1,1)); y = np.where(y_norm < 0.4, 0, np.where(y_norm > 0.6, 1, -1))  # -1 = unlabeled
```

## Evaluation signals

- All normalized endpoint values fall within the 0–1 range for every sample before binarization
- Samples with normalized values < 0.4 are labelled low-activity and those > 0.6 are labelled high-activity; the 0.4–0.6 band is excluded from training
- The same normalization/binarization procedure applied to elastase inhibition, TFC, and TPC yields class distributions compatible with fivefold StratifiedKFold (no empty folds)
- Substituting the binarized labels into the MS2toImg-CNN training loop changes only the label column; the image inputs and hyperparameters remain identical

## Limitations

- Samples falling in the 0.4–0.6 normalized band are implicitly unlabeled, so the label set is a subset of the 545 accessions and the effective sample size depends on the assay value distribution
- The 0.4/0.6 cutoffs are article-specific for DPPH and may not transfer to other bioassay endpoints without re-examining their normalized distributions
- The provided context does not report numeric accuracy values for elastase inhibition, TFC, or TPC (Table 2 values are not in the text), so cross-assay generalizability cannot be independently verified from these excerpts alone

## Evidence

- [other] Normalize each bioassay measurement to a 0-1 range and binarize the labels (for DPPH, normalized values below 0.4 are the low-activity group and values above 0.6 are the high-activity group).: [evidence span withheld — anchor status: not_found]
- [methods] Samples with normalized values below 0.4 were categorized as the low-activity group, while those above 0.6 were categorized as the high-activity group.: "Samples with normalized values below 0.4 were categorized as the low-activity group, while those above 0.6 were categorized as the high-activity group."
- [results] The MS2toImg-CNN model achieved an AUC of 0.96 for discriminating high vs. low DPPH antioxidant activity samples: [evidence span withheld — anchor status: not_found]
- [results] 5-fold cross-validation confusion matrices showing high diagonal values, low standard deviations, and minimal false positives/negatives: [evidence span withheld — anchor status: not_found]
