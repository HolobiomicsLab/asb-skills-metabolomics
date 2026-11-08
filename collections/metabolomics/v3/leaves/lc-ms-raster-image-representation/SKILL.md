---
name: lc-ms-raster-image-representation
description: Use when raw LC-MS peak groups (e.g. MZmine 2 extracted features from
  nontargeted metabolomics runs) must be represented as images to train or apply a
  DNN-based true/false peak classifier.
when_to_use_negative:
- The goal is a purely parametric peak-quality filter using the original 59 peak-group
  parameters (e.g. medRelFWHM) — the article's logistic regression / Random Forest
  models already cover this; the raster-image skill is needed only for the image-based
  DNN path.
- Peak lists are already curated true/false annotations or the peaks lack sample-level
  shape data, so no raster image can be constructed.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3474
tools:
- name: MZmine 2
  role: Generates the non-filtered peak lists whose peak-shape information is converted
    into raster images
- name: EDML_deep_learning2.py
  role: Script that trains the image-based deep neural network on the raster images
    downloaded from the dataset DOI
  repo: https://doi.org/doi:10.25345/C5FD2F
- name: R
  role: Custom R scripts supporting the straightforward implementation alongside MZmine
    2 workflows
  repo: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:b004f6a706215d3b7d091997e893f96c5f79534aa9cc17474eb7217690429890
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - EDML_deep_learning2.py
  - MZmine 2
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# lc-ms-raster-image-representation

## Summary

Converts LC-MS peak group shape information into raster images — each pixel an intensity, each row a sample, each column a retention-time bin — so an image-based deep neural network can classify candidate peaks as true or false.

## When to use

Use when raw LC-MS peak groups (e.g. MZmine 2 extracted features from nontargeted metabolomics runs) must be represented as images to train or apply a DNN-based true/false peak classifier.

## When NOT to use

- The goal is a purely parametric peak-quality filter using the original 59 peak-group parameters (e.g. medRelFWHM) — the article's logistic regression / Random Forest models already cover this; the raster-image skill is needed only for the image-based DNN path.
- Peak lists are already curated true/false annotations or the peaks lack sample-level shape data, so no raster image can be constructed.

## Inputs

- Non-filtered LC-MS peak list generated with MZmine 2 (raw data extracted under More Restrictive or Less Restrictive settings)
- Per-peak-group sample-level peak intensity/shape information (peak information from the 63 samples in which the candidate peak was most intense)
- Datasets downloadable from https://doi.org/doi:10.25345/C5FD2F for training the neural net from scratch

## Outputs

- Raster image per peak group (pixel = intensity, row = one sample, column = retention-time point) suitable for the image-based deep neural network
- DNN probability score of being a true peak per peak group (e.g. calibrated against a threshold of 0.4725)

## How to apply

Generate non-filtered peak lists with MZmine 2, then for each extracted feature build a stacked peak plot from the peak information of the 63 samples in which the candidate peak was most intense. Convert that peak-shape information into a raster image where each pixel value indicates intensity, each row is a single sample, and each column is a different retention-time point. Feed these raster images to the image-based deep neural network (script EDML_deep_learning2.py, after downloading datasets from https://doi.org/doi:10.25345/C5FD2F) so the model assigns each peak group a probability score of being a true peak. The raster form preserves the full peak-shape signal that scalar peak-group parameters (59 total, e.g. medRelFWHM) cannot fully capture, which is why the DNN outperforms logistic-regression and Random-Forest baselines (AUC 0.992 train / 0.988 test).

## Related tools

- **MZmine 2** (Generates the non-filtered peak lists whose peak-shape information is converted into raster images)
- **EDML_deep_learning2.py** (Script that trains the image-based deep neural network on the raster images downloaded from the dataset DOI) — https://doi.org/doi:10.25345/C5FD2F
- **R** (Custom R scripts supporting the straightforward implementation alongside MZmine 2 workflows) — https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks

## Examples

```
python EDML_deep_learning2.py  # after downloading datasets from https://doi.org/doi:10.25345/C5FD2F
```

## Evaluation signals

- Each raster image has rows equal to single samples (up to the 63 most-intense samples) and columns equal to retention-time points, with pixel values encoding intensity
- Model trained on the 1304 manually reviewed training examples (50% true, 50% false) reaches AUC 0.992 on training and 0.988 on the 726-peak test set
- At the selected score threshold 0.4725 the classifier yields ~0.945 true-positive rate and ~0.04 false-positive rate
- Applied to an 11,026-feature restrictive list the DNN captures 98% of true positive peaks while cutting false positives by >50%, and by ~90% under less restrictive settings

## Limitations

- Training-set requirements depend on the specific LC method used; fewer than 1000 peak examples suffice but the exact size varies by method
- Conventional peak selection alone yields roughly 50% true / 50% false peaks under restrictive settings and ~75% false positives under less restrictive settings, so input peak lists are heavily contaminated
- The DNN's probability-score calibration threshold (0.4725) was chosen for this HILIC-based human plasma LC-MS method; re-tuning may be needed for other chromatography setups
- Only proof-of-concept scope: first demonstration of machine-learning LC-MS spectral peak filtering; the article does not name the specific neural network framework used

## Evidence
