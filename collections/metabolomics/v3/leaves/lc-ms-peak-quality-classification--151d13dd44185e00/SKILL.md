---
name: lc-ms-peak-quality-classification
description: Use when you have non-filtered MZmine 2 peak lists from nontargeted LC-MS
  metabolomics data and need to separate true spectral features from random false
  noise peaks prior to downstream analysis.
when_to_use_negative:
- When you only need conventional quality filtering via peak group parameters — the
  paper's simpler alternative is a multiple logistic regression model on 15 peak group
  parameters or a Random Forest on the original 59 parameters, not the image-based
  DNN
- When input is already a curated feature table with no false/noise peaks to remove
- When no manually reviewed peak examples are available for training, calibration,
  and testing
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2
  role: Generates the non-filtered peak lists from raw LC-MS data using more restrictive
    and less restrictive peak extraction parameters before DNN classification
- name: R
  role: Custom R scripts used with MZmine 2 workflows to generate stacked peak plots
    from extracted features
- name: EDML_deep_learning2.py
  role: Author-provided script used to train the image-based deep neural network on
    the downloaded datasets
  repo: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:40fc83083bf76019f5e9e1beb613feb7f0db7108b3bced6cdb5bcad5f2607cda
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
  tools:
  - EDML_deep_learning2.py
  - MZmine 2
  - R
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# lc-ms-peak-quality-classification

## Summary

Classifies nontargeted LC-MS spectral peaks as true vs. false by converting stacked peak plots into 64x64 raster images and scoring them with a calibrated image-based deep neural network, removing up to ~90% of false peaks without losing true positives.

## When to use

Use when you have non-filtered MZmine 2 peak lists from nontargeted LC-MS metabolomics data and need to separate true spectral features from random false noise peaks prior to downstream analysis.

## When NOT to use

- When you only need conventional quality filtering via peak group parameters — the paper's simpler alternative is a multiple logistic regression model on 15 peak group parameters or a Random Forest on the original 59 parameters, not the image-based DNN
- When input is already a curated feature table with no false/noise peaks to remove
- When no manually reviewed peak examples are available for training, calibration, and testing

## Inputs

- Non-filtered peak lists generated with MZmine 2 from raw nontargeted HILIC LC-MS metabolomics data (mzXML-derived)
- Deposited Cohort 1 raw peak data and author-provided manual peak classifications from https://doi.org/doi:10.25345/C5FD2F
- Manually reviewed peak examples: 1304 training (50% true / 50% false), 740 calibration, 726 test

## Outputs

- Per-peak-group probability score of being a true peak
- ROC curves and AUC metrics (paper values: 0.992 training, 0.988 test)
- Filtered peak list with false peaks removed at the selected score threshold (0.4725)
- dnn_peak_classification_metrics.csv of classification metrics

## How to apply

Extract non-filtered peak lists from raw LC-MS data with MZmine 2 using both 'More Restrictive' and 'Less Restrictive' peak extraction parameters. For each extracted feature, generate stacked peak plots using peak information from the 63 samples in which the candidate peak was most intense, then encode these as 64x64 raster images where each pixel value indicates intensity, each row a single sample, and each column a time/scans dimension. Train the convolutional neural network (script EDML_deep_learning2.py from the JainLab repo) on 1304 manually reviewed peak examples (50% true, 50% false), calibrate on 740 examples, and test on 726 held-out examples; the model assigns each peak group a probability score of being a true peak. Select the optimum operating point (score threshold 0.4725 in the paper, giving true positive rate ~0.945 and false positive rate ~0.04) from ROC analysis, then apply the thresholded model to filter full peak lists.

## Related tools

- **MZmine 2** (Generates the non-filtered peak lists from raw LC-MS data using more restrictive and less restrictive peak extraction parameters before DNN classification)
- **R** (Custom R scripts used with MZmine 2 workflows to generate stacked peak plots from extracted features)
- **EDML_deep_learning2.py** (Author-provided script used to train the image-based deep neural network on the downloaded datasets) — https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks

## Examples

```
python EDML_deep_learning2.py  # after downloading datasets from https://doi.org/doi:10.25345/C5FD2F
```

## Evaluation signals

- Reproduced model achieves AUC ~0.992 on the training set and ~0.988 on the held-out test set of Cohort 1 peaks
- Selected score threshold of 0.4725 yields a true positive rate of ~0.945 and a false positive rate of ~0.04
- Applied to the same peak windows, the model captures 98% of true positive peaks while reducing false peak detection by more than 50% (restrictive settings)
- Under less restrictive settings the model filters ~90% of false positives with minimal reduction of true positive peaks
- Raster images conform to the 64x64 encoding: pixel = intensity, row = single sample, column = time/scans

## Limitations

- Requires manually reviewed peak examples (1304 training / 740 calibration / 726 test) to train and calibrate the model
- Training from scratch requires downloading the authors' datasets from https://doi.org/doi:10.25345/C5FD2F; no specific deep learning framework version is named in the manuscript
- Under more restrictive conventional settings only ~50% of captured features are true peaks, so model performance depends on the peak extraction parameters chosen in MZmine 2

## Evidence
