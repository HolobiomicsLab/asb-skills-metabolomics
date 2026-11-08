---
name: training-set-downsampling-learning-curve
description: Use when a manually labeled training corpus (e.g., 1304 reviewed true/false
  LC-MS peaks) must be right-sized before committing to full annotation or retraining
  of a raster-image peak-classification model.
when_to_use_negative:
- Goal is to apply the already-optimized classifier to filter peaks (use the trained
  model with score threshold 0.4725 instead of retraining at multiple sizes)
- The classifier is not an image-based deep neural network trained on raster peak
  images — the downsampling/learning-curve protocol here is tied to that architecture
  and its 1304-peak training corpus
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: R
  role: Custom R scripts used to downsample the training set, retrain/evaluate the
    model, and produce the summary table and learning-curve plot
- name: EDML_deep_learning2.py
  role: Script used to train the neural net model from scratch on the downloaded datasets
    at each subsampled training-set size
  repo: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:6c77e068b1bc9b3a7835ad81d1262569439cbf586b671b2d93759e4758a5d354
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
  tools:
  - EDML_deep_learning2.py
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# training-set-downsampling-learning-curve

## Summary

Quantifies how many manually classified LC-MS peak examples are needed to train the image-based deep neural network peak classifier by retraining on progressively downsampled training sets and plotting a learning curve against calibration/test performance.

## When to use

Use when a manually labeled training corpus (e.g., 1304 reviewed true/false LC-MS peaks) must be right-sized before committing to full annotation or retraining of a raster-image peak-classification model.

## When NOT to use

- Goal is to apply the already-optimized classifier to filter peaks (use the trained model with score threshold 0.4725 instead of retraining at multiple sizes)
- The classifier is not an image-based deep neural network trained on raster peak images — the downsampling/learning-curve protocol here is tied to that architecture and its 1304-peak training corpus

## Inputs

- Manually reviewed peak training examples (1304 peaks, 50% true / 50% false) from https://doi.org/doi:10.25345/C5FD2F
- Raster-image representations of peak groups (pixel = intensity, row = sample, column = retention time)
- 740 calibration peak examples and 726 test peak examples with true/false labels

## Outputs

- Summary table of ROC/AUC at each training-set size
- Learning-curve plot of calibration/test performance versus training-set size
- Estimated minimum training-set size (below 1000 peaks) for high model performance

## How to apply

Load the manually reviewed peak training examples (1304 total, 50% true peaks and 50% false peaks) from the downloaded dataset (https://doi.org/doi:10.25345/C5FD2F). Downsample the training set at a series of increasing sizes, and at each size retrain the image-based deep neural network (script EDML_deep_learning2.py) on the raster-image representations of the subsampled peaks, where each pixel value indicates intensity, each row a different sample, and each column retention time. Evaluate each retrained model on the 740 calibration examples and 726 test examples, recording probability-score performance via ROC/AUC at each threshold. The decision point is the size at which the learning curve plateaus: the article reports that high performance is achievable with fewer than 1000 training peaks, though the required size depends on the specific LC method used.

## Related tools

- **R** (Custom R scripts used to downsample the training set, retrain/evaluate the model, and produce the summary table and learning-curve plot)
- **EDML_deep_learning2.py** (Script used to train the neural net model from scratch on the downloaded datasets at each subsampled training-set size) — https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks

## Examples

```
python EDML_deep_learning2.py  # retrain on each downsampled subset of the 1304 manually reviewed peaks and record calibration/test AUC
```

## Evaluation signals

- Learning curve at full training-set size reproduces the reported AUCs (0.992 training / 0.988 test)
- Performance on the 740 calibration examples remains high at reduced training-set sizes below 1000 peaks
- Performance on the 726 test examples remains high at reduced training-set sizes below 1000 peaks
- Each subsampled training set preserves the 50% true / 50% false peak class balance
- ROC curves are computed across score-threshold cutoffs, so sensitivity/specificity pairs are reported rather than a single operating point

## Limitations

- The required training-set size depends on the specific LC method used, so the <1000-peak conclusion may not transfer to other LC-MS methods
- Training examples are manually reviewed, so the learning curve reflects the cost of manual annotation of true/false peak examples
- AUC (0.992 training / 0.988 test) is the reference ceiling; subsampled models can only be judged against this full-data performance

## Evidence
