---
name: boolean-mask-array-computation-numpy
description: Use when an LA-ICP-MS elemental image (e.g. a 2D NumPy array such as
  the 56Fe channel of a pancreatic tissue microarray scan) must be segmented into
  tissue and background via unsupervised thresholding.
when_to_use_negative:
- Input is a qualitative visual inspection task where no automatic thresholding is
  desired (the paper also supports manual segmentation).
- The image requires multi-level clustering rather than binary tissue/background separation
  — use Pew2's three-level k-means option instead.
- Input data is raw instrument batch data (Agilent MassHunter batch, Perkin Elmer
  Elan .xl, Thermo Qtegra export) that must first be imported and calibrated via Pew2's
  standardization/drift tools.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0091
tools:
- name: Pew2
  role: Exposes the selection dialog offering automatic background/ROI separation
    via built-in image thresholding methods including Otsu
  repo: https://github.com/djdt/pewpew
- name: NumPy
  role: Provides the array library used to compute the Otsu threshold and the boolean
    greater-than mask
- name: pewlib
  role: Python library implementing import/export of the LA-ICP-MS data used in the
    segmentation workflow
  repo: https://github.com/djdt/pewlib
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:46c664cd94c36c33dbc8ddae7a618c741e12563228a902cbd479fc824771045c
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
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# boolean-mask-array-computation-numpy

## Summary

Compute an automatic tissue/background selection mask for an LA-ICP-MS elemental image using NumPy boolean array operations driven by Otsu's thresholding method, as implemented in Pew2's segmentation dialog.

## When to use

Use when an LA-ICP-MS elemental image (e.g. a 2D NumPy array such as the 56Fe channel of a pancreatic tissue microarray scan) must be segmented into tissue and background via unsupervised thresholding.

## When NOT to use

- Input is a qualitative visual inspection task where no automatic thresholding is desired (the paper also supports manual segmentation).
- The image requires multi-level clustering rather than binary tissue/background separation — use Pew2's three-level k-means option instead.
- Input data is raw instrument batch data (Agilent MassHunter batch, Perkin Elmer Elan .xl, Thermo Qtegra export) that must first be imported and calibrated via Pew2's standardization/drift tools.

## Inputs

- 2D elemental image array from an LA-ICP-MS run (e.g. 56Fe channel of a 1.5 mm human pancreatic adenocarcinoma tissue microarray core), as a NumPy array
- optional pre-existing boolean selection mask from a prior segmentation step

## Outputs

- selection_mask.npy — a boolean NumPy array of the same shape as the input image, true for tissue pixels and false for background

## How to apply

Load the elemental image as a 2D NumPy array and compute the Otsu threshold that maximises between-class variance of foreground and background pixels, mirroring the built-in thresholding methods Pew2's selection dialog exposes (Otsu, mean/median binary filters, multi-level k-means). Classify every pixel with a greater-than comparison against the computed threshold to yield a boolean tissue mask. Optionally intersect the new mask with a previously selected region, and optionally restrict the thresholding to currently selected values, matching the dialog's limiting options. Save the boolean array as selection_mask.npy; the mask must be boolean and the same shape as the input image, with the selected region corresponding to the tissue core visible in the 56Fe image.

## Related tools

- **Pew2** (Exposes the selection dialog offering automatic background/ROI separation via built-in image thresholding methods including Otsu) — https://github.com/djdt/pewpew
- **NumPy** (Provides the array library used to compute the Otsu threshold and the boolean greater-than mask)
- **pewlib** (Python library implementing import/export of the LA-ICP-MS data used in the segmentation workflow) — https://github.com/djdt/pewlib

## Examples

```
import numpy as np; img = np.load('fe_image.npy'); t = np.load('threshold.npy'); mask = img > t; np.save('selection_mask.npy', mask)
```

## Evaluation signals

- Output selection_mask.npy is a boolean array with identical shape to the input elemental image
- Selected pixels correspond to the tissue core rather than background, as visualised in the paper's unsupervised segmentation figure for 56Fe
- Mask reproduces the Otsu segmentation panel (Figure 5d) rather than the mean-threshold (c) or three-level k-means (b) panels
- If intersection with a prior mask is requested, the resulting mask equals the logical AND of the two masks
- If limiting to selected values is requested, the threshold is computed only over currently selected pixels

## Limitations

- Segmentation is binary tissue/background; it does not distinguish multiple tissue sub-regions unless the k-means multi-level option is chosen
- Spot-wise data with weak element presence may not support reliable thresholding since segmentation relies on the elemental signal distribution
- Manual laser spot size and speed must be entered beforehand so the image geometry used for segmentation is correct

## Evidence

- [results] Otsu threshold isolation of tissue from background: [evidence span withheld — anchor status: not_found]
- [results] thresholding methods offered in the selection dialog: [evidence span withheld — anchor status: not_found]
- [results] segmentation purpose: "Segmentation is a fundamental imaging analysis tool used to isolate ROIs from the background. This can be performed either manually or automatically using image thresholding"
- [readme] pewlib role in the workflow: [evidence span withheld — anchor status: external_doc]
