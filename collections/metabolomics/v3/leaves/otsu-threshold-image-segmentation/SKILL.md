---
name: otsu-threshold-image-segmentation
description: Use when an LA-ICP-MS elemental image (2D delimited text / NumPy export)
  needs tissue/background segmentation via automatic image thresholding, e.g. Otsu,
  median binary filter, or multi-level k-means.
when_to_use_negative:
- The image is spot-wise or imzML-format data without a contiguous 2D pixel grid —
  Pew2's Otsu selection assumes a standard line-by-line image after laser-log or CSV
  import.
- A fully supervised/manual segmentation is required; Pew2's dialog offers manual
  ROI tracing, and Otsu is an unsupervised method that will fail on images with only
  faint or gradient-like tissue contrast.
- The data has not yet been drift-corrected and spike-filtered in Pew2; raw instrument
  noise can distort the between-class variance Otsu relies on.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0003
tools:
- name: Pew2
  role: Provides the selection dialog that exposes Otsu's method as a binary thresholding
    filter for isolating ROIs from background; mask can be intersected with prior
    selections.
  repo: https://github.com/djdt/pewpew
- name: NumPy
  role: 2D array library used to implement Otsu thresholding and the pixel-wise greater-than
    classification that yields the boolean mask.
- name: pewlib
  role: Python library for importing and exporting the LA-ICP-MS data that becomes
    the 2D image to be segmented.
  repo: https://github.com/djdt/pewlib
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:9d76b3d5e98fa8b8b069e2aae2bd058b5bf5fc6aac519517122b4c53c72bb04b
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - pewlib
  - pewpew
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0003
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# otsu-threshold-image-segmentation

## Summary

Automatic image segmentation of an LA-ICP-MS elemental image (e.g. 56Fe in a human pancreatic adenocarcinoma tissue microarray) using Otsu's thresholding method to separate tissue regions of interest from background, as implemented in Pew2's selection dialog and reproducible with NumPy array programming.

## When to use

Use when an LA-ICP-MS elemental image (2D delimited text / NumPy export) needs tissue/background segmentation via automatic image thresholding, e.g. Otsu, median binary filter, or multi-level k-means.

## When NOT to use

- The image is spot-wise or imzML-format data without a contiguous 2D pixel grid — Pew2's Otsu selection assumes a standard line-by-line image after laser-log or CSV import.
- A fully supervised/manual segmentation is required; Pew2's dialog offers manual ROI tracing, and Otsu is an unsupervised method that will fail on images with only faint or gradient-like tissue contrast.
- The data has not yet been drift-corrected and spike-filtered in Pew2; raw instrument noise can distort the between-class variance Otsu relies on.

## Inputs

- 2D elemental image array exported from LA-ICP-MS data (e.g. NumPy export of Agilent MassHunter data, or delimited text image)
- Calibrated image after import, drift correction, spike removal, and standardization in Pew2
- Optional: an existing boolean selection mask for intersection or 'limit to selected values' thresholding

## Outputs

- Boolean selection mask (same shape as input image) marking tissue pixels vs background, e.g. saved as selection_mask.npy
- Segmented ROI usable for background removal and image feature selection (Figure 5d)

## How to apply

Load the calibrated elemental image as a 2D NumPy array (e.g. the 56Fe image from the 1.5 mm microarray core acquired at 10 μm beam diameter, 40 μm/s scan speed, 0.25 s dwell). Compute the Otsu threshold that maximises between-class variance of foreground and background pixels, then classify each pixel with a greater-than comparison to produce a boolean selection mask. In Pew2's selection dialog this corresponds to choosing the Otsu binary filter; the workflow card mirrors it by writing the mask to selection_mask.npy. The dialog also supports intersecting the new mask with a previously selected region and limiting thresholding to currently selected values, so apply those options when refining an existing ROI rather than starting from scratch. Pew2 additionally offers mean thresholding and three-level k-means as alternative automatic segmentation algorithms for comparison.

## Related tools

- **Pew2** (Provides the selection dialog that exposes Otsu's method as a binary thresholding filter for isolating ROIs from background; mask can be intersected with prior selections.) — https://github.com/djdt/pewpew
- **NumPy** (2D array library used to implement Otsu thresholding and the pixel-wise greater-than classification that yields the boolean mask.)
- **pewlib** (Python library for importing and exporting the LA-ICP-MS data that becomes the 2D image to be segmented.) — https://github.com/djdt/pewlib

## Examples

```
import numpy as np; img = np.load('56Fe.npy'); thr = otsu(img); mask = img > thr; np.save('selection_mask.npy', mask)
```

## Evaluation signals

- Output mask must be a boolean NumPy array with the same shape as the input image.
- Tissue pixels selected by the Otsu mask should correspond to the tissue visible in the 56Fe image, isolating the microarray core from background (Figure 5d).
- Mask covers a contiguous region matching the 1.5 mm tissue core dimensions implied by the 10 μm beam and 40 μm/s scan speed.
- Compare against Pew2's mean-thresholding and three-level k-means alternatives: masks should agree on the bulk tissue core but may differ on edges.
- Verify the mask is not empty (some pixels selected) and not saturated (not all pixels selected), consistent with separating background from tissue.

## Limitations

- Otsu assumes bimodal intensity distribution; Pew2's dialog therefore also provides median binary filters and multi-level k-means for images where two classes are insufficient.
- Peak detection or spike artefacts (instrument noise) can skew the between-class variance; run Pew2's drift-correction and spike-removal filters first.
- Spot-wise import via a single 'marker' element produces data where per-pixel Otsu may not be meaningful without prior re-gridding.

## Evidence

- [results] Unsupervised segmentation of 56Fe using three-level k-means, mean thresholding, and Otsu's method producing masks for background removal and image feature selection: [evidence span withheld — anchor status: not_found]
- [other] Pew2 workflow card: Otsu threshold comparison produces a boolean tissue mask: [evidence span withheld — anchor status: not_found]
