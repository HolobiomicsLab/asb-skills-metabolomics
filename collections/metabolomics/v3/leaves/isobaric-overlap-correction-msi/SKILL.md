---
name: isobaric-overlap-correction-msi
description: Use when centroided MSI lipid ion images contain unresolved isobaric
  overlaps — [X+2:Y+3][M+H]+ overlapped with [X:Y][M+Na]+, or Type II same-class double-bond
  isobars (e.g. PC 34:0 vs PC 34:1) — that cannot be mass-resolved (Δm/z 0.0025 needs
  resolution ≥600,000).
when_to_use_negative:
- When the mass spectrometer can directly resolve the Na/H isobaric Δm/z of 0.0025
  (resolution ≥600,000) or ion-mobility (e.g. timsTOF fleX TIMS) separation is available
  — use resolved images instead of mathematical correction.
- When no co-sprayed class-specific internal standards are available, the sodiated-to-protonated
  adduct ratio (RANa) needed for Na/H deconvolution cannot be estimated.
- When isobaric overlaps involve lipid pairs that are not same-class single-double-bond
  pairs (Type II correction subtracts isotope contributions only within such pairs).
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: LipidQMap
  role: Open-source GUI platform that performs the Na/H adduct and Type II isobaric
    corrections and one-point calibration on .imzML/AnnData/MuData inputs
  repo: https://github.com/swinnenteam/LipidQMap
- name: Cardinal
  role: Source of the HDF5 feature-by-pixel exports (spectraData/intensity, featureData/feature_id)
    whose isobars are compared between ion-mobility-resolved and mathematically corrected
    images
- name: NumPy
  role: Array engine for pixel-wise subtraction of predicted isotope/Na-adduct contributions
    and for writing corrected_img.npy / per-sample pickles
- name: alexandrovteam/pyimzML
  role: Python parser for the imzML 1.1.0 format that LipidQMap ingests for vendor-neutral
    MSI data import
  repo: https://github.com/alexandrovteam/pyimzML
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_002
  source_papers:
  - doi: 10.1101/2025.10.15.682422
    title: LipidQMap - An Open-Source Tool for Quantitative Mass Spectrometry Imaging
      of Lipids
  source_content_hash: sha256:ad6507892d3c4bffa5659fa45970ec9b9c2d3b787706864577277f35773a5dc9
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/swinnenteam/LipidQMap
  tools:
  - Cardinal
  - LipidQMap
  - NumPy
  - alexandrovteam/pyimzML
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# isobaric-overlap-correction-msi

## Summary

LipidQMap's mathematical correction of isobaric overlaps in quantitative MSI lipidomics: Na/H adduct deconvolution using standard-derived sodiated-to-protonated adduct ratios (RANa) and Type II double-bond overlap correction using predicted isotope patterns.

## When to use

Use when centroided MSI lipid ion images contain unresolved isobaric overlaps — [X+2:Y+3][M+H]+ overlapped with [X:Y][M+Na]+, or Type II same-class double-bond isobars (e.g. PC 34:0 vs PC 34:1) — that cannot be mass-resolved (Δm/z 0.0025 needs resolution ≥600,000).

## When NOT to use

- When the mass spectrometer can directly resolve the Na/H isobaric Δm/z of 0.0025 (resolution ≥600,000) or ion-mobility (e.g. timsTOF fleX TIMS) separation is available — use resolved images instead of mathematical correction.
- When no co-sprayed class-specific internal standards are available, the sodiated-to-protonated adduct ratio (RANa) needed for Na/H deconvolution cannot be estimated.
- When isobaric overlaps involve lipid pairs that are not same-class single-double-bond pairs (Type II correction subtracts isotope contributions only within such pairs).

## Inputs

- centroided .imzML MSI files imported by LipidQMap (positive/negative ion mode auto-detected)
- per-sample pickle files containing dict[str, numpy.ndarray] of quantitative ion images and summed-adduct images keyed by species ID
- Cardinal HDF5 exports (spectraData/intensity float32 shape (n_features, n_pixels), featureData/feature_id, pixelData/pixel_index, samples/<index>/height_px and width_px)
- user-modifiable lipid database (.csv / Excel) with lipid_class, adduct, neutral_id columns
- co-sprayed lipid class-specific internal standard signals

## Outputs

- isobar-corrected ion image dictionaries written as one pickle per sample
- corrected feature-by-pixel intensity array as float32 NumPy file (np.save to corrected_img.npy)
- corrected [M+H]+ images feeding downstream quantitation in pmol/mm²

## How to apply

For Na/H correction, load each per-sample pickle (dict[str, numpy.ndarray] of quantitative and summed-adduct ion images keyed by species ID); for each [M+H]+ species, compute the class-specific sodiated-to-protonated ratio (RANa) from co-sprayed class-specific standards, estimate the Na-adduct contribution to the protonated channel pixel-wise, subtract it, skip lipid classes flagged unsupported, and treat NaN summed-adduct pixels as absent. For Type II correction, identify same-class pairs differing by exactly one double bond from the lipid database (featureData with lipid_class, adduct, neutral_id), compute the predicted M, M+1, M+2 isotope pattern of the lower-mass isobar, and subtract its fractional contribution per pixel from the measured intensity of the higher species. The two corrections are applied sequentially, before one-point calibration against class-specific internal standards that produces pmol/mm² maps.

## Related tools

- **LipidQMap** (Open-source GUI platform that performs the Na/H adduct and Type II isobaric corrections and one-point calibration on .imzML/AnnData/MuData inputs) — https://github.com/swinnenteam/LipidQMap
- **Cardinal** (Source of the HDF5 feature-by-pixel exports (spectraData/intensity, featureData/feature_id) whose isobars are compared between ion-mobility-resolved and mathematically corrected images)
- **NumPy** (Array engine for pixel-wise subtraction of predicted isotope/Na-adduct contributions and for writing corrected_img.npy / per-sample pickles)
- **alexandrovteam/pyimzML** (Python parser for the imzML 1.1.0 format that LipidQMap ingests for vendor-neutral MSI data import) — https://github.com/alexandrovteam/pyimzML

## Examples

```
Rscript scripts/h5_to_cardinal.R  # then in Python: import numpy as np; np.save('corrected_img.npy', corrected_features_by_pixels)
```

## Evaluation signals

- Lipid classes flagged unsupported by H/Na overlap correction must remain bit-identical to their input images in the output pickles.
- Corrected protonated images must contain no values propagated from NaN summed-adduct entries, and each output pickle must load as dict[str, numpy.ndarray].
- Corrected intensities must be non-negative for the corrected isobar pairs, and the corrected array shape must equal (n_features, n_pixels) of the source HDF5 with features joined via feature_id and pixels via pixel_index.
- Pixel-wise agreement between ion-mobility-resolved (timsTOF fleX TIMS) and mathematically corrected images — Pearson/Spearman correlation, regression slope/intercept, mean absolute relative deviation — must be finite over the full pixel set (n_pixels = height_px × width_px) with correlations in [-1, 1]; corrected PC 34:1 [M+2+H]+ and PE 38:4 [M+2+H]+ images should closely resemble their [M+H]+ counterparts.

## Limitations

- The Type II correction performs optimally when protonating matrices are used; further optimization is needed for qMSI with various MALDI matrices.
- H/Na overlap correction skips unsupported lipid classes rather than correcting them; identifications with substantial missing pixel data remain unchecked by default.
- Low-abundance signals (e.g. <8000 a.u. on timsTOF instruments) should be avoided for recalibration as they may lead to incorrect signal assignment; recalibration parameters may need adjustment per instrument platform.
- Winsorization settings should preserve 95–99% of pixels (ideal for timsTOF MALDI imaging); the imputation of empty pixels uses the mean of the surrounding 3×3 neighborhood.

## Evidence
