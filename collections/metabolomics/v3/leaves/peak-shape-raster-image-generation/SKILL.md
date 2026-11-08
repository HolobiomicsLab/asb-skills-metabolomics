---
name: peak-shape-raster-image-generation
description: Use when you have MZmine 2-extracted LC-MS peak groups (e.g., from human
  plasma HILIC LC-MS data) and need to represent each peak's shape across samples
  as a raster image for image-based deep learning peak classification.
when_to_use_negative:
- You only need tabular peak-group parameters (e.g., medRelFWHM) for the 15-parameter
  multiple logistic regression or Random Forest models — those operate on peak parameters,
  not raster images.
- Peaks have already been scored/classified by the neural network — raster generation
  is a preprocessing step feeding the model, not a post-processing step.
- Data are not LC-MS spectral peaks with per-sample peak shapes (e.g., already-summarized
  feature intensity tables without chromatographic shape information).
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3474
tools:
- name: MZmine 2
  role: Extracts the raw LC-MS data and generates the non-filtered peak lists whose
    peak shape information is converted into raster images
- name: R
  role: Custom R scripts used alongside common MZmine 2 workflows to generate stacked
    peak plots and raster representations of peak groups
- name: EDML_deep_learning2.py
  role: Script from the manuscript repository used to train/apply the image-based
    deep neural network on the generated raster images
  repo: JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.9b02983
    title: Deep Neural Networks for Classification of LC-MS Spectral Peaks
  source_content_hash: sha256:7d4e08f55f2215d1354e10e20c6aa82eaa8f1a26b10441f6c25c2b35dbda0e0b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks
  tools:
  - Deep neural network (image based)
  - EDML_deep_learning2.py
  - MZmine 2
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# peak-shape-raster-image-generation

## Summary

Converts LC-MS peak shape information from MZmine 2 peak lists into raster images (rows = samples, columns = scan points, pixel values = intensity) so that an image-based deep neural network can classify peak groups as true or false peaks.

## When to use

Use when you have MZmine 2-extracted LC-MS peak groups (e.g., from human plasma HILIC LC-MS data) and need to represent each peak's shape across samples as a raster image for image-based deep learning peak classification.

## When NOT to use

- You only need tabular peak-group parameters (e.g., medRelFWHM) for the 15-parameter multiple logistic regression or Random Forest models — those operate on peak parameters, not raster images.
- Peaks have already been scored/classified by the neural network — raster generation is a preprocessing step feeding the model, not a post-processing step.
- Data are not LC-MS spectral peaks with per-sample peak shapes (e.g., already-summarized feature intensity tables without chromatographic shape information).

## Inputs

- MZmine 2 non-filtered peak lists (more restrictive and/or less restrictive settings) from nontargeted HILIC LC-MS data
- Peak information table giving per-sample intensities/peak shapes for each candidate peak group
- Identifiers of the 63 samples in which each candidate peak was most intense

## Outputs

- Raster images (rows = samples, columns = scan points, pixel values = intensity) for each extracted feature
- Stacked peak plots for each extracted feature (visualization counterpart of the raster images)

## How to apply

For each extracted feature, produce a stacked peak plot using peak information from the 63 samples in which the candidate peak was most intense, matching the representation used for model training. Render this peak shape information as a raster image where each pixel value indicates intensity, each row represents a single sample, and each column represents a different scan point. Generate these images identically for both the more-restrictive and less-restrictive MZmine 2 peak lists so the same trained network (score threshold 0.4725) can be applied. The raster encoding is what lets the deep neural network score each peak group with a probability of being a true peak, enabling ~90% false-peak removal without reducing true positive signals.

## Related tools

- **MZmine 2** (Extracts the raw LC-MS data and generates the non-filtered peak lists whose peak shape information is converted into raster images)
- **R** (Custom R scripts used alongside common MZmine 2 workflows to generate stacked peak plots and raster representations of peak groups)
- **EDML_deep_learning2.py** (Script from the manuscript repository used to train/apply the image-based deep neural network on the generated raster images) — JainLab/Manuscript-DNNs-for-Classification-of-LCMS-Peaks

## Examples

```
download datasets from https://doi.org/doi:10.25345/C5FD2F ; python EDML_deep_learning2.py
```

## Evaluation signals

- Each raster image must have rows corresponding to samples and columns to scan points, with pixel values encoding intensity, matching the representation used to train the network (1304 training examples, 50% true / 50% false).
- Images must be built from peak information in the 63 samples where each candidate peak was most intense; using a different sample subset breaks train/apply consistency.
- Downstream check: applying the trained network at the optimum score threshold of 0.4725 to Cohort 2 (3000 manually classified peak groups, more restrictive settings) should retain ~88% of true peaks and eliminate ~89% of false peaks.
- Reference performance of the consuming model: AUC 0.992 on the training set and 0.988 on the test set; degraded AUC suggests the raster encoding deviates from the training representation.
- Cross-check against the peak-parameter baseline: the parameter model achieves ~80% true retained / ~66% false eliminated; the image-based path should exceed this on false-peak elimination.

## Limitations

- The representation is tied to the paper's LC-MS/HILIC metabolomics context (human plasma, Cohorts 1 and 2); transfer to other chromatography modes or acquisition styles is not validated here.
- The raster is built from only the 63 samples in which the candidate peak was most intense — datasets with fewer samples or different cohort sizes require re-deriving this window.
- No specific software, library, or framework used to build the deep neural network model is named, and EDML_deep_learning2.py is provided without a version, so reproducibility of the image/model pipeline depends on the deposited datasets (doi:10.25345/C5FD2F).

## Evidence
