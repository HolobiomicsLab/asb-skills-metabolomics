---
name: la-icp-ms-image-region-selection
description: Use when you have a 2D LA-ICP-MS elemental image (e.g. 56Fe) loaded as
  a numeric array and need an unsupervised boolean mask separating tissue from background
  for downstream feature selection or background removal.
when_to_use_negative:
- When manual delineation of the ROI is preferred over automatic thresholding — Pew2's
  selection dialog supports both manual and automatic segmentation, and Otsu's method
  assumes a bimodal tissue/background distribution.
- When the image has more than two signal populations that a single threshold cannot
  separate (e.g. multiple tissue compartments); Pew2 offers three-level k-means thresholding
  for such cases instead.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3382
tools:
- name: Pew2
  role: GUI application whose selection dialog implements automatic thresholding (median/Otsu
    binary filters and multiple-level k-means) to segment ROIs from the background
    of LA-ICP-MS images
  repo: https://github.com/djdt/pewpew
- name: NumPy
  role: array library used to represent the 2D elemental image and perform the threshold
    comparison that yields the boolean mask
- name: pewlib
  role: Python library underlying data import/export that supplies the image data
    processed by the selection dialog
  repo: https://github.com/djdt/pewlib
- name: PySide2
  role: Qt5 GUI toolkit used to build the selection dialog interface in Pew2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:35a6c70d24424a2f171b09e753700c07b4aaabb0c0e060107ff838947c23f128
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - PySide2
  - pewlib
  - pewpew
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# la-icp-ms-image-region-selection

## Summary

Isolate regions of interest (ROIs) from the background of a LA-ICP-MS elemental image by thresholding the pixel intensity array, as implemented in Pew2's selection dialog with methods such as Otsu's, mean, and k-means thresholding.

## When to use

Use when you have a 2D LA-ICP-MS elemental image (e.g. 56Fe) loaded as a numeric array and need an unsupervised boolean mask separating tissue from background for downstream feature selection or background removal.

## When NOT to use

- When manual delineation of the ROI is preferred over automatic thresholding — Pew2's selection dialog supports both manual and automatic segmentation, and Otsu's method assumes a bimodal tissue/background distribution.
- When the image has more than two signal populations that a single threshold cannot separate (e.g. multiple tissue compartments); Pew2 offers three-level k-means thresholding for such cases instead.

## Inputs

- 2D NumPy array of elemental image pixel intensities (e.g. calibrated 56Fe channel)
- optional pre-existing boolean selection mask for intersection
- optional limiting option: restrict thresholding to currently selected values

## Outputs

- boolean selection mask (same shape as input image)
- selection_mask.npy file

## How to apply

Load the elemental image as a 2D NumPy array of pixel values (e.g. the calibrated 56Fe channel from a pancreatic adenocarcinoma microarray acquisition). Compute an automatic threshold that maximizes between-class variance (Otsu's method) or use mean/multiple-level k-means thresholding as alternatives offered by Pew2's selection dialog. Compare each pixel value against the computed threshold with a greater-than comparison to generate a boolean selection mask of the same shape as the input image. Optionally intersect the new mask with a previously selected region, or restrict thresholding to only currently selected values, mirroring the dialog's limiting options. Save the resulting boolean array to selection_mask.npy for later background removal or image feature selection.

## Related tools

- **Pew2** (GUI application whose selection dialog implements automatic thresholding (median/Otsu binary filters and multiple-level k-means) to segment ROIs from the background of LA-ICP-MS images) — https://github.com/djdt/pewpew
- **NumPy** (array library used to represent the 2D elemental image and perform the threshold comparison that yields the boolean mask)
- **pewlib** (Python library underlying data import/export that supplies the image data processed by the selection dialog) — https://github.com/djdt/pewlib
- **PySide2** (Qt5 GUI toolkit used to build the selection dialog interface in Pew2)

## Examples

```
import numpy as np
img = np.load('image_56Fe.npy')
mask = img > otsu_threshold(img)
np.save('selection_mask.npy', mask)
```

## Evaluation signals

- The output mask must be a boolean array with exactly the same shape as the input image.
- Masked region should correspond to the tissue visible in the 56Fe image, isolating the tissue core from the surrounding background as demonstrated in Pew2's unsupervised segmentation figure.
- Pixels classified as tissue should have intensities greater than the computed Otsu threshold; verify a greater-than comparison was used.
- If intersection with a prior mask was requested, the final mask must be a subset of that prior mask.
- Saving to selection_mask.npy must round-trip: reloading the file reproduces the same boolean mask.

## Limitations

- Otsu's method assumes a bimodal intensity distribution; multi-compartment images require multiple-level k-means thresholding as offered by Pew2.
- The mask quality depends on prior calibration and drift/spike correction of the image; uncorrected instrument noise can distort the threshold boundary.
- Selection is performed on a single elemental channel; combining channels (e.g. RGB overlay) requires separate masks per element.

## Evidence

- [results] Figure 5. Unsupervised segmentation of 56Fe (a), using three-level k-means (b), mean thresholding (c), and Otsu's method (d). The masks produced by each algorithm can be used for removal: [evidence span withheld — anchor status: not_found]
- [abstract] a Python-based implementation using “NumPy”: [evidence span withheld — anchor status: not_found]
- [methods] Pew2 was written in Python using the “NumPy” and “PySide2” libraries and is available on Git Hub: [evidence span withheld — anchor status: not_found]
- [results] The masks produced by each algorithm can be used for removal of the background and selection of image features.: [evidence span withheld — anchor status: not_found]
