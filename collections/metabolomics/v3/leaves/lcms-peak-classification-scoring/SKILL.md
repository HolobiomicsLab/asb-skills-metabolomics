---
name: lcms-peak-classification-scoring
description: Use when nontargeted LC-MS peak lists (e.g. from MZmine 2 centWave extraction)
  contain a large fraction of false noise peaks and you need to classify and filter
  peak groups using peak-shape information.
when_to_use_negative:
- The input is already a manually curated or conventionally filtered feature table
  where false peaks have already been removed — the skill targets noisy non-filtered
  peak lists.
- The data are not LC-MS peak groups with per-sample peak-shape information (e.g.
  no sample-by-scan-point intensity matrices can be constructed from the 63 most intense
  samples).
- You need direct metabolite structural identification rather than peak quality filtering
  — this skill classifies peaks as true/false only.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MZmine 2
  role: Generates the non-filtered peak lists from raw LC-MS data under 'More Restrictive'
    and 'Less Restrictive' peak extraction parameters that feed the classifier
- name: R
  role: Custom R scripts implement the MZmine 2-integrated peak processing and model-building
    workflow alongside the neural network
- name: Deep neural network (image based)
  role: Trained model that scores each peak group's raster image with a probability
    of being a true peak for threshold-based filtering
  repo: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
- name: EDML_deep_learning2.py
  role: Script used to train the neural net model from scratch on the deposited datasets
  repo: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
- name: Multiple logistic regression
  role: Baseline peak-filtering model built via forward selection from 15 peak group
    parameters for comparison against the image-based DNN
- name: Random Forest
  role: Alternative model developed from the original 59 peak group parameters that
    showed a similar level of performance
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:46e53e33a7b51c4ea2a901448a76e4e3498519735a4488c50f4d75f6ebd34baa
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
  tools:
  - Deep neural network (image based)
  - EDML_deep_learning2.py
  - MZmine 2
  - Multiple logistic regression
  - R
  - Random Forest
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# lcms-peak-classification-scoring

## Summary

Scores LC-MS peak groups as true or false peaks using an image-based deep neural network that treats peak shapes as raster images (rows = samples, columns = scan points, pixel values = intensity), then filters peak lists at a selected score threshold. Validated to remove ~90% of false peaks from nontargeted LC-MS metabolomics data without reducing true positive signals.

## When to use

Use when nontargeted LC-MS peak lists (e.g. from MZmine 2 centWave extraction) contain a large fraction of false noise peaks and you need to classify and filter peak groups using peak-shape information.

## When NOT to use

- The input is already a manually curated or conventionally filtered feature table where false peaks have already been removed — the skill targets noisy non-filtered peak lists.
- The data are not LC-MS peak groups with per-sample peak-shape information (e.g. no sample-by-scan-point intensity matrices can be constructed from the 63 most intense samples).
- You need direct metabolite structural identification rather than peak quality filtering — this skill classifies peaks as true/false only.

## Inputs

- Non-filtered peak lists extracted from raw LC-MS data using MZmine 2 (more restrictive or less restrictive peak extraction parameters)
- Raster/stacked peak plot images per peak group built from peak information of the 63 samples in which the candidate peak was most intense
- Datasets from the deposit at doi:10.25345/C5FD2F (Cohort 1 [N=78] and Cohort 2 [N=526] human plasma HILIC LC-MS data)
- Manually reviewed peak examples for training/calibration/testing (1304 training, 740 calibration, 726 test from Cohort 1)

## Outputs

- Per-peak-group probability scores of being a true peak
- Binary true/false peak classifications at the chosen score threshold (e.g. 0.4725)
- ROC curves with sensitivity and specificity at each threshold cutoff
- Filtered feature list and per-model true-peak retention and false-peak removal rates (e.g. dnn_param_filter_metrics.csv)

## How to apply

Extract non-filtered peak lists from raw LC-MS data with MZmine 2 (e.g. under 'More Restrictive' and 'Less Restrictive' parameters). For each candidate peak group, build a raster image from peak information of the 63 samples in which that peak was most intense — each row is a sample, each column a scan point, and each pixel value the intensity. Score each peak group with the trained image-based deep neural network (e.g. via the EDML_deep_learning2.py script on data from doi:10.25345/C5FD2F), which assigns a probability score of being a true peak. Apply the optimum score threshold of 0.4725 (true positive rate ~0.945, false positive rate ~0.04) to classify peaks, then compute the fraction of true peaks retained and false peaks eliminated; the Cohort 2 evaluation showed ~88% true retention and ~89% false elimination under more restrictive settings. For comparison, a 15-parameter multiple logistic regression model built by forward selection (top parameter medRelFWHM) achieved ~80% true retention and ~66% false elimination, and a Random Forest trained on the original 59 peak group parameters showed similar performance.

## Related tools

- **MZmine 2** (Generates the non-filtered peak lists from raw LC-MS data under 'More Restrictive' and 'Less Restrictive' peak extraction parameters that feed the classifier)
- **R** (Custom R scripts implement the MZmine 2-integrated peak processing and model-building workflow alongside the neural network)
- **Deep neural network (image based)** (Trained model that scores each peak group's raster image with a probability of being a true peak for threshold-based filtering) — https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
- **EDML_deep_learning2.py** (Script used to train the neural net model from scratch on the deposited datasets) — https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
- **Multiple logistic regression** (Baseline peak-filtering model built via forward selection from 15 peak group parameters for comparison against the image-based DNN)
- **Random Forest** (Alternative model developed from the original 59 peak group parameters that showed a similar level of performance)

## Examples

```
python EDML_deep_learning2.py  # after downloading the datasets from https://doi.org/doi:10.25345/C5FD2F
```

## Evaluation signals

- On the Cohort 2 evaluation set (3000 manually classified peak groups, more restrictive MZmine 2 settings), the DNN at threshold 0.4725 must retain ~88% of true peaks and eliminate ~89% of false peaks relative to the conventional baseline (11,026 features, ~50% true / ~50% false).
- The 15-parameter logistic regression comparison must yield ~80% true retained and ~66% false eliminated, matching the results section.
- ROC analysis must reproduce AUC of 0.992 in the training set and 0.988 in the test set, with the optimum threshold 0.4725 corresponding to TPR ~0.945 and FPR ~0.04.
- Overall false-peak filtering improvement should reach ~90% under less restrictive settings with minimal reduction of true positive peaks.
- Output must be a summary table (dnn_param_filter_metrics.csv) with per-model true-peak retention and false-peak removal rates.

## Limitations

- The model and evaluation are grounded in human plasma HILIC LC-MS data (Cohorts 1 and 2); transfer to other chromatography modes or matrices is not demonstrated here.
- No specific software, library, or framework used to build the deep neural network model is named in the manuscript, and no version details are given for the neural net implementation (the script EDML_deep_learning2.py is provided without a version).
- Performance depends on the optimum score threshold selection (0.4725); applying a different threshold shifts the true positive/false positive trade-off.
- Manual review of peak examples is required for model training (1304 training, 740 calibration, 726 test examples).

## Evidence
