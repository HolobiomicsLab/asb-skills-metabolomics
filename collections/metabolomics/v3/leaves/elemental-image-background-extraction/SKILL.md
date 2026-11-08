---
name: elemental-image-background-extraction
description: Use when a 2D LA-ICP-MS elemental image (e.g. a calibrated 56Fe, 55Mn,
  63Cu, or 66Zn NumPy array) must be split into tissue and background before quantification
  or mask-based feature selection.
when_to_use_negative:
- Input is already a labeled mask or a three-level RGB overlay — thresholding would
  re-segment already-classified pixels rather than extract background.
- Peak detection or drift correction is the actual goal; Pew2 separates segmentation
  from its spike-removal and polynomial drift tools.
- The sample has no distinct tissue/background contrast (e.g. homogeneous gelatin
  standard), where Otsu between-class variance has no meaningful separation.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: Pew2
  role: Provides the selection dialog implementing binary filters (median, Otsu) and
    multi-level k-means thresholding for automatic segmentation of ROIs from background.
  repo: https://github.com/djdt/pewpew
- name: NumPy
  role: Holds the 2D elemental image array that is thresholded; masks are boolean
    NumPy arrays of the same shape.
- name: pewlib
  role: Python library for importing/processing LA-ICP-MS data that can be scripted
    to produce the image array fed into thresholding.
  repo: https://github.com/djdt/pewlib
- name: pewpew
  role: Command-line entry point for Pew2 after pip install -e . from the cloned repository.
  repo: https://github.com/djdt/pewpew
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:14d98b91a1ed125858d01218c537b24139d52d40f67709605e0471388fd20bad
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - pewlib
  - pewpew
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Elemental image background extraction

## Summary

Automatically segments an LA-ICP-MS elemental image into tissue/ROI and background by applying image thresholding — notably Otsu's method, mean thresholding, or multi-level k-means — as implemented in Pew2's selection dialog.

## When to use

Use when a 2D LA-ICP-MS elemental image (e.g. a calibrated 56Fe, 55Mn, 63Cu, or 66Zn NumPy array) must be split into tissue and background before quantification or mask-based feature selection.

## When NOT to use

- Input is already a labeled mask or a three-level RGB overlay — thresholding would re-segment already-classified pixels rather than extract background.
- Peak detection or drift correction is the actual goal; Pew2 separates segmentation from its spike-removal and polynomial drift tools.
- The sample has no distinct tissue/background contrast (e.g. homogeneous gelatin standard), where Otsu between-class variance has no meaningful separation.

## Inputs

- 2D calibrated elemental image array (e.g. 56Fe, 55Mn, 66Zn) exported from Pew2 or pewlib as a NumPy array
- optional existing boolean selection mask for intersection
- optional subset of currently selected values to restrict the thresholding

## Outputs

- Boolean selection mask (same shape as input image) identifying tissue/ROI pixels
- npy file such as selection_mask.npy containing the mask

## How to apply

Load the elemental image as a 2D NumPy array, then compute a threshold that separates signal from background using one of Pew2's built-in binary filters: Otsu's method (maximizes between-class variance), mean thresholding, or a multi-level k-means for more than two classes. Classify each pixel by a greater-than comparison against the computed threshold to produce a boolean selection mask, mirroring Pew2's selection dialog options. The dialog also supports intersecting the new mask with a previously selected region and limiting the thresholding method to only currently selected values, which lets you restrict segmentation to a sub-region. Save the resulting boolean mask (e.g. selection_mask.npy) for background removal or image-feature selection downstream. Validate that the mask is boolean, has the same shape as the input image, and that selected pixels correspond to the visible tissue core — e.g. Otsu on the 56Fe pancreatic adenocarcinoma TMA image should isolate the 1.5 mm tissue core from the gelatin/background signal.

## Related tools

- **Pew2** (Provides the selection dialog implementing binary filters (median, Otsu) and multi-level k-means thresholding for automatic segmentation of ROIs from background.) — https://github.com/djdt/pewpew
- **NumPy** (Holds the 2D elemental image array that is thresholded; masks are boolean NumPy arrays of the same shape.)
- **pewlib** (Python library for importing/processing LA-ICP-MS data that can be scripted to produce the image array fed into thresholding.) — https://github.com/djdt/pewlib
- **pewpew** (Command-line entry point for Pew2 after pip install -e . from the cloned repository.) — https://github.com/djdt/pewpew

## Examples

```
import numpy as np; from skimage.filters import threshold_otsu; img = np.load('56Fe.npy'); mask = img > threshold_otsu(img); np.save('selection_mask.npy', mask)
```

## Evaluation signals

- Mask array dtype is bool and shape equals the input image shape.
- Otsu threshold applied to the 56Fe pancreatic TMA image selects pixels coinciding with the visible 1.5 mm tissue core, excluding gelatin/background.
- Mask intersections with a prior selection do not exceed the prior selection's extent.
- Three-level k-means produces a mask distinct from two-class Otsu/mean thresholding, consistent with Figure 5's unsupervised segmentation comparison.
- Resulting mask, when used for background removal, leaves calibrated element concentrations (e.g. Table 2 coefficients: 56Fe gradient 18.71) unchanged in selected pixels.

## Limitations

- Thresholding is unsupervised; masks 'can be used for removal' of background but rely on the image having clear tissue/background contrast, as shown for 56Fe.
- Requires at least one element present across the entire sample for reliable segmentation, per Pew2's spot-wise import constraint.
- The article demonstrates Otsu and mean thresholding on a single pancreatic adenocarcinoma TMA core; behavior on low-contrast or heterogeneous samples is not validated.

## Evidence

- [results] Pew2's segmentation tool offers automatic background/ROI separation via built-in image thresholding methods, including binary filters such as median and Otsu's method, as well as multiple-level k-means thresholding.: "Segmentation is a fundamental imaging analysis tool used to
isolate ROIs from the background. This can be performed
either manually or automatically using image thresholding"
- [results] Unsupervised segmentation of 56Fe demonstrated using three-level k-means, mean thresholding, and Otsu's method producing masks for background removal and image feature selection.: "Unsupervised segmentation of 56Fe (a), using three-level k-
means (b), mean thresholding (c), and Otsu ’ s method (d). The
masks produced by each algorithm can be used for removal"
- [methods] Otsu thresholding isolates the tissue core from background in the 56Fe pancreatic adenocarcinoma TMA image.: "a 1.5 mm human pancreatic adenocarcinoma tissue microarray core (US Biomax; Rockville, MD) was imaged using a NWR193 laser ablation system"
- [readme] Pew2 is a GUI for importing and processing LA-ICP-MS data using the python library pewlib, installable from the djdt/pewpew repository.: "Pew² is a GUI for importing and processing line-by-line, spot-wise and ablation-time-aligned LA-ICP-MS data using the python library [pewlib]"
- [results] Spot-wise data import requires at least one element present across the entire sample for peak integration, a prerequisite for reliable segmentation.: "requires at least one element to be present across the entirety
of the sample and for the positions of peaks for each element
to be similar."
