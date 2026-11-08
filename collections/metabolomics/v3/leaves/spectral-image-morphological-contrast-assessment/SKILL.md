---
name: spectral-image-morphological-contrast-assessment
description: Use when you have a centroid-mode peak matrix from an MSI dataset where
  matrix-related signals were annotated/removed and you need to demonstrate post-processing
  improvement via visual PCA exploration of morphological structure contrast.
when_to_use_negative:
- Input is raw profile-mode spectra or an imzML file rather than a rMSIproc-generated
  centroid-mode peak matrix (rMSIcleanup requires the rMSIproc data format)
- You need a quantitative contrast metric rather than the paper's qualitative visual
  comparison of morphological structures
- The data do not contain matrix-related peaks (no LDI-promoting material such as
  a silver nanolayer was applied), so there is nothing to annotate and remove
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: rMSIcleanup
  role: Annotates matrix-related signals by chemical formula and spatial distribution
    and removes them from the peak matrix prior to PCA
  repo: https://github.com/gbaquer/rMSIcleanup
- name: rMSIproc
  role: Generates and loads the centroid-mode peak matrix consumed by the PCA comparison
  repo: https://github.com/prafols/rMSIproc
- name: rMSI
  role: Loads the full processed MSI dataset (.tar) used alongside the peak matrix
    for annotation
  repo: https://github.com/prafols/rMSIproc
- name: R
  role: Implementation language; stats::prcomp computes PCA on original and cleaned
    peak matrices with centring and no scaling
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-020-00449-0
    title: 'rMSIcleanup: an open-source tool for matrix-related peak annotation in
      mass spectrometry imaging and its application to silver-assisted laser desorption/ionization'
  source_content_hash: sha256:97dd07ae7fedce325cf38c6188fba447fa7c23ce697e8027082c48cb9b637712
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gbaquer/rMSIcleanup
  tools:
  - R
  - rMSI
  - rMSIcleanup
  - rMSIproc
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spectral-image-morphological-contrast-assessment

## Summary

Qualitative PCA-based visual assessment of whether annotating and removing matrix-related peaks (e.g. Agn+ clusters in silver-assisted LDI MSI) improves spectral data post-processing, by rendering PC1–PC3 score images as RGB composites before versus after peak removal and judging morphological contrast enhancement.

## When to use

Use when you have a centroid-mode peak matrix from an MSI dataset where matrix-related signals were annotated/removed and you need to demonstrate post-processing improvement via visual PCA exploration of morphological structure contrast.

## When NOT to use

- Input is raw profile-mode spectra or an imzML file rather than a rMSIproc-generated centroid-mode peak matrix (rMSIcleanup requires the rMSIproc data format)
- You need a quantitative contrast metric rather than the paper's qualitative visual comparison of morphological structures
- The data do not contain matrix-related peaks (no LDI-promoting material such as a silver nanolayer was applied), so there is nothing to annotate and remove

## Inputs

- Peak matrix in rMSIproc format (.zip), e.g. Ag_Pancreas_TOF_2015_Dataset2.zip
- Processed MSI dataset in rMSI format (.tar), e.g. Ag_Pancreas_TOF_2015_Dataset2_proc.tar
- Matrix chemical formula for annotation, e.g. 'Ag1'

## Outputs

- RGB composite image comparing PC1–PC3 score images before and after matrix peak removal (pca_before_after_comparison.png)
- Table of first three principal component scores per pixel for both conditions (pca_scores_table.csv)

## How to apply

Load the peak matrix (rMSIproc::LoadPeakMatrix on the .zip, e.g. 'Ag_Pancreas_TOF_2015_Dataset2.zip') and the full processed data (rMSI::LoadMsiData on the .tar). Obtain the cleaned matrix via rMSIcleanup::annotate_matrix with the matrix chemical formula (e.g. 'Ag1') followed by rMSIcleanup::remove_matrix. Compute PCA with stats::prcomp on both original and cleaned peak matrices, centring the data and applying no scaling, consistent with the physical relationship of MSI features; no data normalization is performed. Extract the first three principal component scores per pixel for both conditions, encode PC1, PC2, PC3 as the red, green and blue channels of an RGB composite image for each condition, and judge correctness by confirming a generalized enhancement in the contrast of morphological structures after removal of Agn+ peaks.

## Related tools

- **rMSIcleanup** (Annotates matrix-related signals by chemical formula and spatial distribution and removes them from the peak matrix prior to PCA) — https://github.com/gbaquer/rMSIcleanup
- **rMSIproc** (Generates and loads the centroid-mode peak matrix consumed by the PCA comparison) — https://github.com/prafols/rMSIproc
- **rMSI** (Loads the full processed MSI dataset (.tar) used alongside the peak matrix for annotation) — https://github.com/prafols/rMSIproc
- **R** (Implementation language; stats::prcomp computes PCA on original and cleaned peak matrices with centring and no scaling)

## Examples

```
pks<-rMSIproc::LoadPeakMatrix("Ag_Pancreas_TOF_2015_Dataset2.zip"); full<-rMSI::LoadMsiData("Ag_Pancreas_TOF_2015_Dataset2_proc.tar"); res<-rMSIcleanup::annotate_matrix(pks,"Ag1",full); pks_clean<-rMSIcleanup::remove_matrix(pks,res); prcomp(t(pks_clean), center=TRUE, scale.=FALSE)
```

## Evaluation signals

- Qualitative visual comparison of the RGB composites shows a generalized enhancement in the contrast of morphological structures across the first principal components after matrix peak removal
- Both prcomp runs use centring and no scaling, and no data normalization is applied, matching the paper's stated processing
- The cleaned matrix contains only peaks removed by rMSIcleanup annotation (formula 'Ag1'); all other peaks are unchanged
- Output artifacts pca_before_after_comparison.png and pca_scores_table.csv are produced with PC1, PC2 and PC3 encoded as red, green and blue channels respectively

## Limitations

- Validation is qualitative (visual inspection of contrast enhancement) rather than quantitative
- No scaling must be applied in PCA, so results depend on the unnormalised, unscaled feature space; no data normalization was performed
- Only the first three principal components are rendered; later components are not assessed

## Evidence

- [abstract] Visual exploration of the data using Principal Component Analysis (PCA) demonstrated that annotation and removal of matrix-related signals improved spectral data post-processing.: "Visual exploration of the data using Principal Component Analysis (PCA) demonstrated that annotation and removal of matrix-related signals improved spectral data post-processing."
- [methods] Removal of matrix-related peaks enhances morphological contrast: "the removal of matrix-related peaks leads to a generalized enhancement in the contrast of morphological structures obtained with the first principal components"
- [methods] No data normalization was performed.: "No data normalization was performed."
- [readme] Sample data download requirements: "You need to download both the Peak Matrix ("Ag_Pancreas_TOF_2015_Dataset2.zip") and the Processed File ("Ag_Pancreas_TOF_2015_Dataset2_proc.tar")."
