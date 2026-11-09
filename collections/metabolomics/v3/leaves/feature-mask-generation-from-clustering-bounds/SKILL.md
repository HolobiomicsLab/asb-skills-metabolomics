---
name: feature-mask-generation-from-clustering-bounds
description: Use when an LA-ICP-MS elemental image (e.g., 56Fe in a 1.5 mm TMA core
  imported as an Agilent MassHunter batch) needs an objective, unsupervised ROI mask
  without manual threshold tuning, and the input supports automatic image thresholding
  or multi-level clustering.
when_to_use_negative:
- When the user has already identified ROIs by manual delineation or has a previously
  produced mask — applying automatic thresholding adds unneeded bias risk.
- When the elemental image is dominated by instrument noise or unremoved spikes, since
  the local mean/median spike-removal filter should be applied first for the segmentation
  to be meaningful.
- When a fully supervised classification or an existing labeled segmentation already
  exists — k-means clustering is unsupervised and may not match domain labels without
  validation.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3361
tools:
- name: Pew2
  role: Provides the calculator tool in which the built-in kmeans and mask functions
    are executed to turn clustering bounds into an ROI mask.
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library implementing LA-ICP-MS import/export used to get vendor data
    (Agilent, Perkin Elmer, Thermo) into a NumPy form for segmentation.
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Array backend on which the calibrated image and bounds array are held; also
    the format for the precalibrated supplementary export.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:e4cd9cb5edb1c23933b9482a257e23b0fe64cb8d019d211bb6fee58a12c25593
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - Python
  - pewlib
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# feature-mask-generation-from-clustering-bounds

## Summary

Pew2's unsupervised segmentation skill: convert a multi-level (e.g., k-means) clustering bounds array of an LA-ICP-MS elemental image (such as 56Fe over a pancreatic adenocarcinoma TMA core) into a binary mask that isolates regions of interest from background, for downstream background removal and image feature selection.

## When to use

Use when an LA-ICP-MS elemental image (e.g., 56Fe in a 1.5 mm TMA core imported as an Agilent MassHunter batch) needs an objective, unsupervised ROI mask without manual threshold tuning, and the input supports automatic image thresholding or multi-level clustering.

## When NOT to use

- When the user has already identified ROIs by manual delineation or has a previously produced mask — applying automatic thresholding adds unneeded bias risk.
- When the elemental image is dominated by instrument noise or unremoved spikes, since the local mean/median spike-removal filter should be applied first for the segmentation to be meaningful.
- When a fully supervised classification or an existing labeled segmentation already exists — k-means clustering is unsupervised and may not match domain labels without validation.

## Inputs

- Calibrated LA-ICP-MS elemental image loaded into Pew2 (e.g., a 56Fe channel from an Agilent MassHunter batch, after calibration, drift removal, and spike removal)
- Desired number of clusters for multi-level segmentation (e.g., k = 3 for three-level k-means)
- Optional: precalibrated NumPy export of the image (supplied in the article's supplementary material)

## Outputs

- Binary mask image (per pixel/spot) isolating ROIs from background, usable for background removal and image feature selection
- Bounds array from the k-means segmentation whose levels give cluster thresholds
- Segmented/feature-selected image channel (e.g., Figure 5(b) of the article)

## How to apply

Load the calibrated elemental image into the Pew2 calculator tool and run the built-in kmeans function over the chosen isotope channel (e.g., segment(Fe56, kmeans(Fe56, 3)) for three clusters). The k-means routine returns a bounds array whose levels correspond to cluster thresholds; select the highest k level from that bounds array and mask the original data with it (e.g., mask(Fe56, masked == 3)) to produce a binary mask for background removal and feature selection. Pew2 also offers median and Otsu's method binary filters as alternative automatic thresholding routes when k-means clustering is unsuitable. Segmentation is accepted when the resulting mask visually separates tissue from background on the isotope image without manual threshold adjustment, matching the mask pattern reported in the paper's Figure 5(b) for the microarray core.

## Related tools

- **Pew2** (Provides the calculator tool in which the built-in kmeans and mask functions are executed to turn clustering bounds into an ROI mask.) — https://github.com/djdt/pewpew
- **pewlib** (Python library implementing LA-ICP-MS import/export used to get vendor data (Agilent, Perkin Elmer, Thermo) into a NumPy form for segmentation.) — https://github.com/djdt/pewlib
- **NumPy** (Array backend on which the calibrated image and bounds array are held; also the format for the precalibrated supplementary export.)

## Examples

```
In Pew2 calculator: segment(Fe56, kmeans(Fe56, 3)); mask(Fe56, masked == 3)
```

## Evaluation signals

- The three-level k-means mask (analogous to Figure 5(b)) visually separates tissue from background on the 56Fe image without any manual threshold tuning.
- The mask is derived from the highest k level of the returned bounds array (e.g., masked == 3 when k = 3), not from an arbitrary hand-set threshold.
- Applying the mask removes background signal from the isotope channel while retaining the structurally isolated ROI pixels, consistent with the stated purpose of masks for background removal and image feature selection.
- Running alternative automatic thresholders (mean thresholding, Otsu's method) on the same channel yields masks comparable to the k-means result, indicating the segmentation is stable across built-in methods.
- The mask dimensions match the original Pew2 image after correct laser spot size/speed entry (10 μm beam, 40 μm/s), confirming the segmentation applied to a properly configured image.

## Limitations

- The k-means approach is unsupervised and cluster count must be chosen by the user; masks from different cluster counts may not align with tissue labels.
- Segmentation quality depends on prior drift removal and spike removal — unfiltered instrumental artifacts can skew cluster boundaries.
- Automatic thresholding methods (median, Otsu, k-means) may disagree; the article notes masks produced by each algorithm can differ and must be evaluated for the specific application.

## Evidence

- [results] Unsupervised segmentation of 56Fe demonstrated using three-level k-means, mean thresholding, and Otsu's method producing masks for background removal and image feature selection: [evidence span withheld — anchor status: not_found]
- [results] Pew2 calculator tool supporting kmeans and mask functions: [evidence span withheld — anchor status: not_found]
