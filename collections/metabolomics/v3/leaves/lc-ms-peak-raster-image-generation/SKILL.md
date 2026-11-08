---
name: lc-ms-peak-raster-image-generation
description: Use when non-targeted LC-MS peak lists (e.g., MZmine 2 mzXML-derived
  features) must be transformed into per-peak raster images to feed an image-based
  CNN for true/false peak classification.
when_to_use_negative:
- The goal is parameter-based peak filtering (multiple logistic regression or Random
  Forest on the 59 peak group parameters) rather than image-based CNN classification
- Input is already an image or feature tensor; this skill converts raw peak shape
  data, not pre-encoded representations
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: MZmine 2
  role: Extracts non-filtered peak lists from raw LC-MS data that supply the per-feature
    peak shape information for the stacked plots
- name: R
  role: Custom R scripts generate the stacked peak plots and convert them to raster
    images for the deep neural network
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:6206dd64bfbca56e1c33cf9312cf881ff05e91f2cabf5c82d0fb2ec25da36659
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - MZmine 2
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# lc-ms-peak-raster-image-generation

## Summary

Converts LC-MS peak group shape information into 64x64 raster images (stacked peak plots encoded as pixel intensities) so that image-based deep neural networks can classify peaks as true or false. It is the feature-representation step that bridges MZmine 2 peak extraction and CNN-based peak filtering.

## When to use

Use when non-targeted LC-MS peak lists (e.g., MZmine 2 mzXML-derived features) must be transformed into per-peak raster images to feed an image-based CNN for true/false peak classification.

## When NOT to use

- The goal is parameter-based peak filtering (multiple logistic regression or Random Forest on the 59 peak group parameters) rather than image-based CNN classification
- Input is already an image or feature tensor; this skill converts raw peak shape data, not pre-encoded representations

## Inputs

- Raw LC-MS data (mzXML-derived) processed into non-filtered peak lists by MZmine 2
- Peak group shape/intensity information per extracted feature across samples
- Manually reviewed peak classifications (Cohort 1: 1304 training, 740 calibration, 726 test examples) from https://doi.org/doi:10.25345/C5FD2F

## Outputs

- 64x64 raster image per candidate peak group, pixel value = intensity, row = sample, column = time/scans
- Stacked peak plots built from the 63 samples where the peak was most intense

## How to apply

First extract non-filtered peak lists from raw mzXML data with MZmine 2. For each extracted feature, generate a stacked peak plot using peak information from only the 63 samples in which the candidate peak was most intense. Convert the stacked peak shape information into a raster image in which each pixel value encodes intensity, each row represents a single sample, and each column represents the time/scans dimension, producing 64x64 images readable by the deep neural network. These images then serve as the training/calibration/test inputs (1304 training, 740 calibration, 726 test manually reviewed Cohort 1 peak examples) for the CNN script EDML_deep_learning2.py.

## Related tools

- **MZmine 2** (Extracts non-filtered peak lists from raw LC-MS data that supply the per-feature peak shape information for the stacked plots)
- **R** (Custom R scripts generate the stacked peak plots and convert them to raster images for the deep neural network)

## Evaluation signals

- Each generated raster image is 64x64 pixels, with pixel intensity encoding signal, rows corresponding to individual samples, and columns to the time/scans dimension
- Each image is built from exactly the 63 samples in which the candidate peak was most intense
- One raster image is produced per extracted peak group in the MZmine 2 non-filtered peak list
- Downstream check: the CNN trained on these images reproduces AUC of ~0.992 (training) and ~0.988 (test) on Cohort 1

## Limitations

- Requires manually reviewed peak labels and Cohort 1 data deposited at https://doi.org/doi:10.25345/C5FD2F to train/evaluate; the script EDML_deep_learning2.py is provided without version details
- The image representation was developed and validated on HILIC-based LC-MS human plasma metabolomics data (Cohort 1 N=78, Cohort 2 N=526); transfer to other chromatography modes is not demonstrated here

## Evidence
