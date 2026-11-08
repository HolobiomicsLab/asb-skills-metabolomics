---
name: rgb-principal-component-image-compositing
description: Use when you have a centroided MSI peak matrix (e.g. rMSIproc .zip) and
  need an exploratory, per-pixel spatial visualization of dominant variance sources
  — e.g. to compare data quality before vs after peak filtering or cleaning.
when_to_use_negative:
- The input is raw profile-mode spectra or an imzML file rather than a processed centroided
  peak matrix — preprocess with rMSIproc::ProcessWizard first.
- The goal is quantitative biomarker discovery or statistical testing rather than
  qualitative visual exploration of morphological contrast.
- The peak matrix has already been cleaned of matrix-related peaks and no before/after
  comparison is intended.
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: rMSIproc
  role: Loads the centroided peak matrix (LoadPeakMatrix) whose pixel intensities
    are decomposed by PCA and stores the before/after peak matrices
  repo: https://github.com/prafols/rMSIproc
- name: rMSI
  role: Loads the processed MSI dataset (.tar) to supply pixel coordinates and spatial
    context for rendering the PC score images
- name: rMSIcleanup
  role: Provides the matrix-cleaned peak matrix (annotate_matrix/remove_matrix) used
    as the 'after' condition in the PCA before/after RGB comparison
  repo: https://github.com/gbaquer/rMSIcleanup
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-020-00449-0
    title: 'rMSIcleanup: an open-source tool for matrix-related peak annotation in
      mass spectrometry imaging and its application to silver-assisted laser desorption/ionization'
  source_content_hash: sha256:eb1c6a63e8a54ccebfce43779262e4e9c9cbc6076d999dae1fdbdaa17f9b2eab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/prafols/rMSIproc
  tools:
  - R
  - rMSI
  - rMSIcleanup
  - rMSIproc
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# rgb-principal-component-image-compositing

## Summary

Compute PCA on an MSI peak matrix and encode the first three principal component scores per pixel as the red, green and blue channels of a spatial RGB composite image, as used in the rMSIcleanup paper's Figure 3 to visually demonstrate that removing matrix-related Ag peaks enhances morphological contrast.

## When to use

Use when you have a centroided MSI peak matrix (e.g. rMSIproc .zip) and need an exploratory, per-pixel spatial visualization of dominant variance sources — e.g. to compare data quality before vs after peak filtering or cleaning.

## When NOT to use

- The input is raw profile-mode spectra or an imzML file rather than a processed centroided peak matrix — preprocess with rMSIproc::ProcessWizard first.
- The goal is quantitative biomarker discovery or statistical testing rather than qualitative visual exploration of morphological contrast.
- The peak matrix has already been cleaned of matrix-related peaks and no before/after comparison is intended.

## Inputs

- Centroided MSI peak matrix readable by rMSIproc::LoadPeakMatrix (e.g. Ag_Pancreas_TOF_2015_Dataset2.zip)
- Processed rMSI-format MSI dataset (.tar) loaded via rMSI::LoadMsiData (optional, for spatial metadata)

## Outputs

- RGB composite image with PC1/PC2/PC3 encoded as red/green/blue per pixel (e.g. pca_before_after_comparison.png)
- Per-pixel PC score table for the first three components (e.g. pca_scores_table.csv)

## How to apply

Load the peak matrix with rMSIproc::LoadPeakMatrix (and the processed .tar data with rMSI::LoadMsiData if spatial context from the full dataset is needed). Run PCA on the peak-intensity matrix with stats::prcomp, centring the data but applying no scaling, consistent with the physical relationship of MSI features; apply no data normalization. Extract the first three PC score images (PC1, PC2, PC3) per pixel and map them to the red, green and blue channels of an RGB composite. If the skill is used for a before/after comparison (e.g. of rMSIcleanup::remove_matrix output), compute PCA identically on both matrices so that visual differences reflect the peak removal, not differing preprocessing.

## Related tools

- **rMSIproc** (Loads the centroided peak matrix (LoadPeakMatrix) whose pixel intensities are decomposed by PCA and stores the before/after peak matrices) — https://github.com/prafols/rMSIproc
- **rMSI** (Loads the processed MSI dataset (.tar) to supply pixel coordinates and spatial context for rendering the PC score images)
- **rMSIcleanup** (Provides the matrix-cleaned peak matrix (annotate_matrix/remove_matrix) used as the 'after' condition in the PCA before/after RGB comparison) — https://github.com/gbaquer/rMSIcleanup

## Examples

```
pks <- rMSIproc::LoadPeakMatrix("Ag_Pancreas_TOF_2015_Dataset2.zip"); pca <- prcomp(pks$matrix, center = TRUE, scale. = FALSE)
```

## Evaluation signals

- PCA is fitted with centring enabled and no scaling and no normalization, matching the physical relationship of MSI features.
- The PC score table contains exactly three score columns (PC1–PC3) with one row per pixel, for both the original and cleaned matrices.
- Qualitative check: the 'after' RGB composite shows a generalized enhancement in the contrast of morphological structures relative to the 'before' composite.
- Both PCA fits use identical preprocessing (same matrix format, same pixels) so differences in the composites are attributable to matrix-peak removal.

## Limitations

- Validation is qualitative visual comparison of RGB composites, not a numeric metric.
- No data normalization is applied, so results depend on the raw intensity distribution of the peak matrix.
- Matrix-related peaks (e.g. silver clusters) can dominate early PCs; without prior annotation/removal the composite may mainly reflect the matrix signal.

## Evidence

- [abstract] PCA visual exploration showed improved post-processing: "Visual exploration of the data using Principal Component Analysis (PCA) demonstrated that annotation and removal of matrix-related signals improved spectral data post-processing."
- [methods] Removal of matrix peaks enhances PC contrast: "the removal of matrix-related peaks leads to a generalized enhancement in the contrast of morphological structures obtained with the first principal components"
- [readme] Loading the peak matrix in R: "pks<-rMSIproc::LoadPeakMatrix("[Full Path to Peak Matrix (.zip)]")
full<-rMSI::LoadMsiData("[Full Path to Processed Data (.tar)]")"
