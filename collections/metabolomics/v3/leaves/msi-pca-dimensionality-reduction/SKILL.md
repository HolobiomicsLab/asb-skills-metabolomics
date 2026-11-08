---
name: msi-pca-dimensionality-reduction
description: Use when you have a centroid-mode MSI peak matrix (e.g. rMSIproc .zip
  format) and need dimensionality reduction to visually compare data quality before
  and after a peak-removal step such as matrix-related signal removal.
when_to_use_negative:
- Input is already a processed feature table with non-MSI features whose physical
  relationships would justify unit-variance scaling (scaling was deliberately omitted
  here)
- The dataset is profile-mode raw spectra in imzML rather than a centroid-mode peak
  matrix; rMSIproc processing (smoothing, calibration, peak-picking) must be applied
  first
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: rMSIcleanup
  role: Annotates matrix-related peaks (e.g. Ag1 formula) and removes them from the
    peak matrix to produce the cleaned matrix compared by PCA
  repo: https://github.com/gbaquer/rMSIcleanup
- name: rMSIproc
  role: Loads and stores the peak matrix (.zip) consumed by the PCA analysis
  repo: https://github.com/prafols/rMSIproc
- name: rMSI
  role: Loads the processed MSI dataset (.tar) and supports data visualization/exploration
  repo: https://github.com/prafols/rMSI
- name: R
  role: Implementation environment; PCA computed with stats::prcomp
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-020-00449-0
    title: 'rMSIcleanup: an open-source tool for matrix-related peak annotation in
      mass spectrometry imaging and its application to silver-assisted laser desorption/ionization'
  source_content_hash: sha256:efba4a50daef9854bc679421b981243c11c9ebd74f307cacc62949184b045d9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gbaquer/rMSIcleanup
  tools:
  - R
  - rMSI
  - rMSIcleanup
  - rMSIproc
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# msi-pca-dimensionality-reduction

## Summary

Apply PCA (e.g. stats::prcomp) to mass spectrometry imaging peak matrices, centring data without scaling or normalization, and render the first three principal component scores per pixel as spatial RGB ion images to qualitatively assess spectral post-processing quality.

## When to use

Use when you have a centroid-mode MSI peak matrix (e.g. rMSIproc .zip format) and need dimensionality reduction to visually compare data quality before and after a peak-removal step such as matrix-related signal removal.

## When NOT to use

- Input is already a processed feature table with non-MSI features whose physical relationships would justify unit-variance scaling (scaling was deliberately omitted here)
- The dataset is profile-mode raw spectra in imzML rather than a centroid-mode peak matrix; rMSIproc processing (smoothing, calibration, peak-picking) must be applied first

## Inputs

- Centroid-mode MSI peak matrix in rMSIproc .zip format (e.g. Ag_Pancreas_TOF_2015_Dataset2.zip)
- Processed MSI dataset in rMSI TAR format (e.g. Ag_Pancreas_TOF_2015_Dataset2_proc.tar)
- Optional: cleaned peak matrix produced by rMSIcleanup::remove_matrix for before/after comparison

## Outputs

- RGB composite image of PC1/PC2/PC3 scores per pixel for each condition (e.g. pca_before_after_comparison.png)
- Table of first three principal component scores per pixel (e.g. pca_scores_table.csv)

## How to apply

Load the peak matrix with rMSIproc::LoadPeakMatrix and the corresponding processed dataset with rMSI::LoadMsiData, then apply any peak removal (e.g. rMSIcleanup::annotate_matrix with formula 'Ag1' followed by rMSIcleanup::remove_matrix). Compute PCA on both the original and cleaned peak matrices with stats::prcomp, centring the data but applying no scaling, consistent with the physical relationship of MSI features, and applying no data normalization. Extract the first three principal component (PC1, PC2, PC3) scores per pixel for each condition and render them as spatial ion images. Encode PC1, PC2 and PC3 as the red, green and blue channels of an RGB composite image per condition, mirroring the paper's Figure 3 presentation. Judge success by qualitative visual comparison: removal of the Agn+ matrix peaks should yield a generalized enhancement in the contrast of morphological structures across the first principal components.

## Related tools

- **rMSIcleanup** (Annotates matrix-related peaks (e.g. Ag1 formula) and removes them from the peak matrix to produce the cleaned matrix compared by PCA) — https://github.com/gbaquer/rMSIcleanup
- **rMSIproc** (Loads and stores the peak matrix (.zip) consumed by the PCA analysis) — https://github.com/prafols/rMSIproc
- **rMSI** (Loads the processed MSI dataset (.tar) and supports data visualization/exploration) — https://github.com/prafols/rMSI
- **R** (Implementation environment; PCA computed with stats::prcomp)

## Examples

```
pks <- rMSIproc::LoadPeakMatrix('Ag_Pancreas_TOF_2015_Dataset2.zip'); pca <- prcomp(pks$peakMatrix, center=TRUE, scale.=FALSE); scores <- pca$x[,1:3]
```

## Evaluation signals

- PCA is computed with center=TRUE and scale.=FALSE; confirm no data normalization was applied before PCA
- Exactly the first three principal components are extracted as per-pixel scores and mapped to RGB channels (PC1=red, PC2=green, PC3=blue)
- Qualitative visual comparison shows generalized enhancement of contrast in morphological structures in the cleaned-data PCs versus the original-data PCs
- Both conditions (before and after matrix peak removal) are PCA-computed on identically structured peak matrices so the comparison is like-for-like

## Limitations

- Validation is qualitative (visual comparison), not a quantitative metric; improvement is judged by enhanced contrast of morphological structures in the first PCs
- No scaling is applied, so results depend on absolute intensity relationships between m/z features; large-intensity features dominate the PCs
- No data normalization was performed in this workflow, which the authors state explicitly

## Evidence

- [full_text] PCA no-scaling rationale: [evidence span withheld — anchor status: not_found]
- [methods] PCA contrast finding: "the removal of matrix-related peaks leads to a generalized enhancement in the contrast of morphological structures obtained with the first principal components"
- [abstract] PCA visual exploration finding: "Visual exploration of the data using Principal Component Analysis (PCA) demonstrated that annotation and removal of matrix-related signals improved spectral data post-processing."
- [readme] Data loading commands: "pks<-rMSIproc::LoadPeakMatrix("[Full Path to Peak Matrix (.zip)]")
full<-rMSI::LoadMsiData("[Full Path to Processed Data (.tar)]")"
