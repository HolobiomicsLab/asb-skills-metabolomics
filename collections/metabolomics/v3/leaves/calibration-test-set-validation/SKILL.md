---
name: calibration-test-set-validation
description: Use when training a supervised peak classifier (e.g., image-based DNN
  on raster-image peak plots) and you must estimate generalization performance, pick
  an operating score threshold, or determine the minimum number of manually reviewed
  training peaks needed.
when_to_use_negative:
- You only have a single pooled peak set with no held-out calibration/test split —
  using training AUC alone would inflate performance estimates
- Peak lists have not yet been converted to the raster-image representation required
  by the image-based neural network (e.g., you only have raw MZmine 2 peak parameter
  tables for the logistic-regression/Random Forest baselines)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: R
  role: Custom R scripts used to downsample the training set, evaluate probability-score
    performance (ROC/AUC) on calibration and test sets, and produce learning-curve
    plots and summary tables
- name: EDML_deep_learning2.py
  role: Script used to train the image-based deep neural network model from scratch
    on the downloaded datasets
  repo: JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:c5cb70e7ca639307ec9ef4a94c702704b829db990befaac496192dc945afa618
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
  tools:
  - EDML_deep_learning2.py
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# calibration-test-set-validation

## Summary

Validates an LC-MS peak-classification model by holding out dedicated calibration (740 examples) and test (726 examples) sets, computing ROC/AUC on each, and optionally varying training-set size to show how much manually reviewed data is required. Grounded in the image-based deep neural network workflow for LC-MS spectral peak classification (Kantz et al., Anal. Chem. 2019).

## When to use

Use when training a supervised peak classifier (e.g., image-based DNN on raster-image peak plots) and you must estimate generalization performance, pick an operating score threshold, or determine the minimum number of manually reviewed training peaks needed.

## When NOT to use

- You only have a single pooled peak set with no held-out calibration/test split — using training AUC alone would inflate performance estimates
- Peak lists have not yet been converted to the raster-image representation required by the image-based neural network (e.g., you only have raw MZmine 2 peak parameter tables for the logistic-regression/Random Forest baselines)

## Inputs

- 1304 manually reviewed peak training examples (50% true peaks, 50% false peaks) as raster-image representations of stacked peak plots
- 740 calibration peak examples from Cohort 1
- 726 test peak examples from Cohort 1
- Datasets downloadable from https://doi.org/doi:10.25345/C5FD2F

## Outputs

- ROC curves and AUC values for training (target 0.992), calibration, and test (target 0.988) sets
- Optimized score threshold (e.g., 0.4725) with associated true positive rate (~0.945) and false positive rate (~0.04)
- Learning-curve plot and summary table of calibration/test performance versus training-set size
- Trained deep neural network model (via EDML_deep_learning2.py) assigning each peak group a probability score of being a true peak

## How to apply

Split the manually reviewed peak examples into a training set (1304 examples, 50% true and 50% false peaks from Cohort 1), a calibration set (740 examples), and a test set (726 examples). Train the image-based deep neural network on raster images in which each pixel value indicates intensity, each row a single sample, and each column a retention-time (or m/z) position, using the script EDML_deep_learning2.py on data downloaded from https://doi.org/doi:10.25345/C5FD2F. Score every peak group with the probability of being a true peak, then compute ROC curves on calibration and test sets, recording AUC (expected ~0.992 training / ~0.988 test) and selecting an operating threshold (0.4725 gave TPR ~0.945 and FPR ~0.04). For sample-size questions, downsample the 1304 training examples at a series of increasing sizes, retrain at each size, and evaluate on the same 740 calibration and 726 test examples to build a learning curve showing high performance is achievable with fewer than 1000 training peaks. Use calibration-set performance for threshold/model selection and report held-out test-set performance as the unbiased estimate.

## Related tools

- **R** (Custom R scripts used to downsample the training set, evaluate probability-score performance (ROC/AUC) on calibration and test sets, and produce learning-curve plots and summary tables)
- **EDML_deep_learning2.py** (Script used to train the image-based deep neural network model from scratch on the downloaded datasets) — JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks

## Examples

```
python EDML_deep_learning2.py  # after downloading datasets from https://doi.org/doi:10.25345/C5FD2F
```

## Evaluation signals

- Reproduced test-set AUC is comparable to the reported 0.988 (training 0.992)
- At the selected threshold 0.4725, true positive rate is ~0.945 and false positive rate is ~0.04
- Learning curve remains high at reduced training-set sizes; high performance is achievable with fewer than 1000 training peaks
- Calibration and test splits are disjoint from the 1304-example training set (740 + 726 held-out examples)
- Model outputs a probability score per peak group that yields a valid ROC curve over threshold cutoffs

## Limitations

- The required training-set size depends on the specific LC method used; <1000 peaks may not transfer to other chromatographic methods
- Training examples must be manually reviewed and balanced (~50% true / ~50% false); biased labeling will distort ROC/AUC estimates
- No specific neural-network framework or version is named in the article; the script EDML_deep_learning2.py is provided without version details

## Evidence

- [results] 1304 manually reviewed peak examples used for model training (50% true peaks and 50% false peaks), 740 peak examples used for calibration and 726 peak examples used for testing: "1304 manually reviewed peak examples used for model training (50% true peaks and 50% false peaks), 740 peak examples used for calibration and 726 peak examples used for testing"
- [results] AUC 0.992 / 0.988 training and test: "The curves illustrate an exceptional high specificity and sensitivity with an area under the curve (AUC) of 0.992 and 0.988 in training and test sets, respectively"
- [results] Threshold 0.4725 with TPR ~0.945: "optimum operating point (score threshold) of 0.4725 was selected (see Supporting Information for selection criteria) corresponding to a true positive rate of approximately 0.945"
- [results] Raster image representation for the DNN: "the same information can be represented as a raster image in which each pixel value indicates intensity, each row represents a different single sample"
- [readme] Download datasets and train with EDML_deep_learning2.py: "first download the datasets from https://doi.org/doi:10.25345/C5FD2F. Then use the script EDML_deep_learning2.py."
