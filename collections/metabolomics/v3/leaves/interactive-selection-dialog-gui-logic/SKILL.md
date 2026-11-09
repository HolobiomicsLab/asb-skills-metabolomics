---
name: interactive-selection-dialog-gui-logic
description: Use when a boolean ROI mask must be derived from an LA-ICP-MS elemental
  image (e.g. a 2D NumPy array of a 56Fe pancreatic adenocarcinoma tissue microarray
  core) by automatic thresholding, and the mask must optionally be combined with or
  restricted to a previously selected region.
when_to_use_negative:
- The image is already a labeled or categorical segmentation and no further background/ROI
  thresholding is needed.
- The user needs manual (paint-style) ROI selection rather than automatic image thresholding
  — the article states segmentation 'can be performed either manually or automatically
  using image thresholding' and this skill only implements the automatic path.
- The input is a full multi-channel or RGB overlay image (e.g. 55Mn/56Fe/66Zn RGB
  overlay) where per-element masks must be computed separately rather than a single
  combined mask.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Pew2
  role: Hosts the selection dialog GUI whose logic (automatic image thresholding with
    intersection/limit-to-selection options) is reconstructed by this skill.
  repo: https://github.com/djdt/pewpew
- name: NumPy
  role: 'Library used for the underlying array operations: loading the image, computing
    the Otsu threshold, the greater-than pixel classification, and the boolean mask
    intersection.'
- name: pewlib
  role: Import/export library for LA-ICP-MS data that feeds the 2D elemental images
    into the selection workflow.
  repo: https://github.com/djdt/pewlib
- name: pewpew
  role: The Pew2 GUI application (written with NumPy and PySide2) available on GitHub
    that exposes the selection dialog to users.
  repo: https://github.com/djdt/pewpew
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:2f81fe23e604b5c8720599f251145c36200db0ffc1d5e682d1ea074227f60fd5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - pewlib
  - pewpew
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# interactive-selection-dialog-gui-logic

## Summary

Implements the logic behind Pew2's interactive selection dialog, which isolates regions of interest (ROIs) from the background of an LA-ICP-MS elemental image via automatic image thresholding methods such as Otsu's method, mean thresholding, multi-level k-means, and binary filters like median, optionally intersecting the new mask with an existing selection.

## When to use

Use when a boolean ROI mask must be derived from an LA-ICP-MS elemental image (e.g. a 2D NumPy array of a 56Fe pancreatic adenocarcinoma tissue microarray core) by automatic thresholding, and the mask must optionally be combined with or restricted to a previously selected region.

## When NOT to use

- The image is already a labeled or categorical segmentation and no further background/ROI thresholding is needed.
- The user needs manual (paint-style) ROI selection rather than automatic image thresholding — the article states segmentation 'can be performed either manually or automatically using image thresholding' and this skill only implements the automatic path.
- The input is a full multi-channel or RGB overlay image (e.g. 55Mn/56Fe/66Zn RGB overlay) where per-element masks must be computed separately rather than a single combined mask.

## Inputs

- 2D NumPy array elemental image (e.g. 56Fe channel of a quantified LA-ICP-MS acquisition from a 1.5 mm human pancreatic adenocarcinoma tissue microarray core, 0.25 s acquisition time, 10 µm beam)
- Existing boolean selection mask (optional, for intersection or limiting the thresholding computation)

## Outputs

- Boolean selection mask array of the same shape as the input image (e.g. saved as selection_mask.npy) marking tissue/ROI pixels

## How to apply

Load the elemental image as a 2D NumPy array (the format produced by Pew2's import of Agilent MassHunter batches, Perkin Elmer Elan 'XL', or Thermo Fischer Qtegra exports). Compute a threshold that separates tissue from background using one of the selection dialog's methods — Otsu's method (maximizing between-class variance), mean thresholding, three-level k-means, or median-based binary filtering — mirroring Figure 5's unsupervised segmentation of 56Fe. Compare every pixel against the computed threshold with a greater-than comparison to yield a boolean tissue mask. If the dialog's intersection option is set, AND the new mask with a previously selected region; if the 'limit to current selection' option is set, restrict the thresholding computation to only the currently selected pixel values. Save the resulting same-shape boolean array (e.g. selection_mask.npy) for downstream removal of background or image feature selection.

## Related tools

- **Pew2** (Hosts the selection dialog GUI whose logic (automatic image thresholding with intersection/limit-to-selection options) is reconstructed by this skill.) — https://github.com/djdt/pewpew
- **NumPy** (Library used for the underlying array operations: loading the image, computing the Otsu threshold, the greater-than pixel classification, and the boolean mask intersection.)
- **pewlib** (Import/export library for LA-ICP-MS data that feeds the 2D elemental images into the selection workflow.) — https://github.com/djdt/pewlib
- **pewpew** (The Pew2 GUI application (written with NumPy and PySide2) available on GitHub that exposes the selection dialog to users.) — https://github.com/djdt/pewpew

## Examples

```
import numpy as np; img = np.load('fe_56_image.npy'); from pewlib.threshold import otsu; t = otsu(img); mask = img > t; np.save('selection_mask.npy', mask)
```

## Evaluation signals

- The output mask must be a boolean NumPy array with shape identical to the input elemental image.
- For the 56Fe demonstration, Otsu thresholding should isolate the tissue core (visible in Figure 5d) from background — the mask should correspond to the tissue region shown in the unsupervised segmentation figure.
- If intersection is enabled, the final mask must be a strict subset of the pre-existing selection mask (no newly selected pixels outside the old selection).
- If 'limit to current selection' is enabled, the threshold value must be computed only from pixel values inside the current selection, not the full image.
- The threshold value must lie between the minimum and maximum pixel intensities of the (possibly restricted) image, and the greater-than comparison must produce a non-empty and non-total mask.

## Limitations

- Automatic thresholding methods (Otsu, mean, k-means, median filter) assume a bimodal or multi-modal intensity distribution; they may fail when tissue and background signals are not cleanly separated in a given element channel.
- The selection logic operates per-element (single-channel) image; combining masks across multiple elements (e.g. 55Mn, 56Fe, 66Zn) requires additional co-localization analysis.
- Spot-wise data requires at least one element to be present across the entirety of the sample with similar peak positions for each element, which constrains which images can be reliably thresholded.
- The dialog's limiting options (intersection, restrict-to-selection) depend on a valid pre-existing selection mask; none is available on first use, so the first mask is computed over the whole image.

## Evidence

- [results] Unsupervised segmentation of 56Fe demonstrated using three-level k-means, mean thresholding, and Otsu's method producing masks for background removal and image feature selection: [evidence span withheld — anchor status: not_found]
- [methods] Pew2 was written in Python using the “NumPy” and “PySide2” libraries and is available on Git Hub (https://github.com/djdt/pewpew): [evidence span withheld — anchor status: not_found]
