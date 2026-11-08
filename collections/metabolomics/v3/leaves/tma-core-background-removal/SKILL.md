---
name: tma-core-background-removal
description: Use when a LA-ICP-MS elemental image (e.g., a 1.5 mm TMA core imaged
  at 10 µm beam, 40 µm/s) needs automatic tissue-vs-background separation without
  manual threshold tuning, using multi-level k-means segmentation.
when_to_use_negative:
- The elemental image already contains a clean binary tissue mask or the user prefers
  manual thresholding on a clearly separated signal.
- The dataset is not LA-ICP-MS line-by-line data importable into Pew2 (e.g., already
  a preprocessed feature table or a non-imaging spectral file).
- Only a single cluster level is needed and simpler thresholding (mean or Otsu) would
  suffice, making multi-level k-means unnecessary.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Pew2
  role: Provides the built-in k-means segmentation (kmeans function) and masking operators
    used to isolate ROIs from background and remove background signal from the elemental
    image.
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Underlying Python library that implements import and export of LA-ICP-MS data
    (including Agilent MassHunter batches) used to load the TMA core image prior to
    segmentation.
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Backend array engine for Pew2's image data; precalibrated NumPy exports of
    the microarray core can be loaded directly for segmentation.
  repo: https://github.com/djdt/pewpew
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:62775923aee99074f9860229a481fdabe5682197b470a2b0259ffde0b5d79eb5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - Python
  - pewlib
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# tma-core-background-removal

## Summary

Isolates regions of interest (ROIs) from background in LA-ICP-MS elemental images—such as a 1.5 mm pancreatic adenocarcinoma tissue microarray core—by applying multi-level k-means (plus mean or Otsu thresholding) segmentation within Pew2, then masking the elemental data (e.g., 56Fe) to the highest k level for background removal and feature selection.

## When to use

Use when a LA-ICP-MS elemental image (e.g., a 1.5 mm TMA core imaged at 10 µm beam, 40 µm/s) needs automatic tissue-vs-background separation without manual threshold tuning, using multi-level k-means segmentation.

## When NOT to use

- The elemental image already contains a clean binary tissue mask or the user prefers manual thresholding on a clearly separated signal.
- The dataset is not LA-ICP-MS line-by-line data importable into Pew2 (e.g., already a preprocessed feature table or a non-imaging spectral file).
- Only a single cluster level is needed and simpler thresholding (mean or Otsu) would suffice, making multi-level k-means unnecessary.

## Inputs

- Raw Agilent MassHunter batch (.b directory) containing LA-ICP-MS line-by-line data for a tissue microarray core (e.g., 56Fe, 55Mn, 66Zn isotope channels)
- Elemental image already imported into Pew2 with correct laser spot size (10 µm) and scan speed (40 µm/s) entered manually

## Outputs

- Binary mask (mask(Fe56, masked == 3)) selecting the highest k-means cluster as the ROI, ready for background removal
- Segmented image with background removed and structural features isolated for downstream feature selection

## How to apply

Import the raw Agilent MassHunter batch for the TMA core into Pew2 (drag-and-drop of the .b directory) and confirm the 56Fe isotope channel; after import, manually enter laser spot size (10 µm) and scan speed (40 µm/s) to set the correct image aspect and scale bar. Load the chosen elemental image into the Pew2 calculator tool and run the built-in k-means segmentation with three clusters: segment(Fe56, kmeans(Fe56, 3)). Extract the highest k level from the k-means bounds array and mask the data to it with mask(Fe56, masked == 3), yielding a binary mask for background removal and image feature selection. Pew2 also offers mean thresholding and Otsu's method as alternative unsupervised segmentation algorithms; the k-means path is preferred when multiple tissue levels must be distinguished. Rationale: Pew2's segmentation tool is designed to isolate ROIs from the background automatically, minimizing user bias; masks from any of the algorithms can be applied for removal of background signal.

## Related tools

- **Pew2** (Provides the built-in k-means segmentation (kmeans function) and masking operators used to isolate ROIs from background and remove background signal from the elemental image.) — https://github.com/djdt/pewpew
- **pewlib** (Underlying Python library that implements import and export of LA-ICP-MS data (including Agilent MassHunter batches) used to load the TMA core image prior to segmentation.) — https://github.com/djdt/pewlib
- **NumPy** (Backend array engine for Pew2's image data; precalibrated NumPy exports of the microarray core can be loaded directly for segmentation.) — https://github.com/djdt/pewpew

## Examples

```
segment(Fe56, kmeans(Fe56, 3)); mask(Fe56, masked == 3)
```

## Evaluation signals

- The three-level k-means mask (Figure 5(b)) separates tissue from background on the 56Fe image without manual threshold tuning, matching the mask pattern reported for the microarray core.
- The mask produced by kmeans is a binary array of the same shape as the input elemental image, with the highest k level (masked == 3) marking the ROI.
- Scale bar and image aspect in Pew2 correctly reflect the manually entered laser spot size (10 µm) and scan speed (40 µm/s) before segmentation.
- The segmented output can be applied for background removal and image feature selection, consistent with the role described for masks from each segmentation algorithm.
- The same k-means segmentation applied to a precalibrated NumPy export of the microarray core reproduces the Figure 5(b) result.

## Limitations

- Spot-wise data import requires at least one element present across the entire sample with similar peak positions for each element; if this is not met, segmentation may be unreliable.
- Peak detection may fail when background signal is not uniform, although Pew2 offers windowed thresholding and continuous wavelet transforms for such cases.
- K-means cluster count (e.g., k=3) must be chosen by the user; there is no automatic selection of the optimal number of tissue levels in the described workflow.

## Evidence

- [results] Unsupervised segmentation of 56Fe using three-level k-means, mean thresholding, and Otsu's method producing masks for background removal and image feature selection.: [evidence span withheld — anchor status: not_found]
- [methods] Pew2 was written in Python using the NumPy and PySide2 libraries and is available on GitHub.: [evidence span withheld — anchor status: not_found]
- [methods] The laser was operated at 40 Hz with a 10 µm beam diameter and scan speed of 40 µm s−1.: [evidence span withheld — anchor status: not_found]
- [readme] Drag-and-drop import of batches (.b directories), with one sample per line.: [evidence span withheld — anchor status: external_doc]
