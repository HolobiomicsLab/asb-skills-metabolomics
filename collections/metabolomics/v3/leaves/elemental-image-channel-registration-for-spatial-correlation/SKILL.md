---
name: elemental-image-channel-registration-for-spatial-correlation
description: Use when two or more elemental image channels of identical pixel dimensions
  from a LA-ICP-MS acquisition need quantitative colocalisation statistics, or when
  a region-of-interest (lasso/rectangle) selection must restrict the coefficient inputs.
when_to_use_negative:
- The input consists of single-channel images without a second channel to correlate
  against — colocalisation statistics require at least two co-registered elemental
  maps.
- The two channels have different pixel dimensions or spatial registration; Pew2 requires
  both channels to share identical pixel dimensions before computing coefficients.
- The data are unprocessed raw vendor files that have not yet been imported, calibrated
  (gelatin standards), or drift/spike-corrected — performing colocalisation on uncorrected
  images will misrepresent the spatial relationship.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3344
tools:
- name: Pew2
  role: GUI application hosting the colocalisation dialog that computes Li ICQ, Pearson
    R with ρ, and Manders coefficients on selected elemental image channels
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library used to import and expose the elemental image data as NumPy
    arrays for the colocalisation calculations
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Vectorized arithmetic backend for implementing Li ICQ and Pearson R computations
    over the image arrays
- name: PySide2
  role: Qt5 GUI framework providing the image and selection context-menu entry points
    and the copyable results dialog
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:a04485fef55278328d76c1e7a6227dddcb52c9e447e8121b6ee951df8d53923a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - PySide2
  - Python
  - pewlib
  - pewpew
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3344
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# elemental-image-channel-registration-for-spatial-correlation

## Summary

Quantify the spatial relationship between two LA-ICP-MS elemental image channels (e.g. 55Mn and 66Zn) by computing Li's ICQ, Pearson's R with a shuffle-based probability ρ, and Manders overlap coefficients using the Costes thresholding method.

## When to use

Use when two or more elemental image channels of identical pixel dimensions from a LA-ICP-MS acquisition need quantitative colocalisation statistics, or when a region-of-interest (lasso/rectangle) selection must restrict the coefficient inputs.

## When NOT to use

- The input consists of single-channel images without a second channel to correlate against — colocalisation statistics require at least two co-registered elemental maps.
- The two channels have different pixel dimensions or spatial registration; Pew2 requires both channels to share identical pixel dimensions before computing coefficients.
- The data are unprocessed raw vendor files that have not yet been imported, calibrated (gelatin standards), or drift/spike-corrected — performing colocalisation on uncorrected images will misrepresent the spatial relationship.

## Inputs

- Two or more LA-ICP-MS elemental image channels sharing identical pixel dimensions (e.g. 55Mn and 66Zn from a pancreatic adenocarcinoma tissue microarray core)
- Optional lasso or rectangle region-of-interest selection restricting the coefficient computation to a subregion
- Imported vendor data (Agilent MassHunter batch, Perkin Elmer Elan .xl, Thermo Qtegra export, text image, file-per-line, imzML) previously calibrated and segmented in Pew2

## Outputs

- Li ICQ coefficient for the full image or selected ROI
- Pearson's R correlation coefficient with an associated probability ρ computed via Monte Carlo comparison against random shuffles of the two channels
- Manders overlap coefficients computed using the Costes method
- Copyable results output displayed in the colocalisation dialog

## How to apply

Load the imported LA-ICP-MS elemental image channels (e.g. 55Mn and 66Zn) into Pew2's NumPy-array backend via pewlib, ensuring both channels share identical pixel dimensions before computing any coefficient. Implement the Li ICQ coefficient over the entire image using NumPy vectorized arithmetic as encoded in the colocalisation dialog. Compute Pearson's R over the entire image with its associated probability (ρ) by comparing the R value to those of random shuffles of the two channels. Implement Manders overlap coefficients using the Costes method (doi 10.1529/biophysj.103.038422) for threshold-dependent colocalisation quantification. Wire both the image and selection context-menu entry points so that a lasso or rectangle selection restricts the coefficient inputs to that region only. Validate that computed Pearson R and Li ICQ values on the Mn/Zn channel pair reproduce the positive correlative spatial relationship shown in Table 3 and the magenta-overlay colocalisation in Figure 6.

## Related tools

- **Pew2** (GUI application hosting the colocalisation dialog that computes Li ICQ, Pearson R with ρ, and Manders coefficients on selected elemental image channels) — https://github.com/djdt/pewpew
- **pewlib** (Python library used to import and expose the elemental image data as NumPy arrays for the colocalisation calculations) — https://github.com/djdt/pewlib
- **NumPy** (Vectorized arithmetic backend for implementing Li ICQ and Pearson R computations over the image arrays)
- **PySide2** (Qt5 GUI framework providing the image and selection context-menu entry points and the copyable results dialog)

## Examples

```
pip install pewlib; from pewlib import *; mn, zn = load('55Mn.npy'), load('66Zn.npy'); pearson_r = np.corrcoef(mn.ravel(), zn.ravel())[0,1]
```

## Evaluation signals

- Computed Pearson R and Li ICQ values on the 55Mn/66Zn channel pair must reproduce a positive correlative spatial relationship consistent with the magenta-overlay colocalisation visible in the RGB composite of 55Mn (red), 56Fe (green), and 66Zn (blue).
- The ρ probability must be produced by comparing the Pearson R to those of random shuffles of the two channels (Monte Carlo), triggered by a dedicated button in the dialog.
- Manders overlap coefficients must match the Costes method (doi 10.1529/biophysj.103.038422) and reflect threshold-dependent colocalisation rather than a simple intensity-weighted overlap.
- The results must be presented as a copyable output in the colocalisation dialog's left-side display.
- Restricting the computation via a lasso/rectangle selection must change the coefficient values relative to the whole-image calculation, confirming the ROI constraint is honored.

## Limitations

- Spot-wise data import requires at least one element to be present across the entirety of the sample and for the positions of peaks for each element to be similar, so colocalisation between channels from such acquisitions depends on this shared-marker constraint.
- The probability ρ calculation is a Monte Carlo shuffle-based procedure and must be explicitly triggered via a button rather than computed automatically alongside Pearson R.
- Coefficient computation is restricted to images with identical pixel dimensions; channels with mismatched geometry cannot be compared without prior resampling.

## Evidence

- [results] Pew2 quantifies the spatial overlap of two elemental images by calculating Pearson's R correlation coefficient from the pixel values of the two images: [evidence span withheld — anchor status: not_found]
- [results] Load the imported LA-ICP-MS elemental image channels (e.g. 55Mn and 66Zn) into the Pew2 NumPy-array backend using pewlib so both channels share identical pixel dimensions: [evidence span withheld — anchor status: not_found]
- [results] the colocalization of Mn and Zn is clearly visible as magenta regions within the image: "The colocalization of Mn and Zn is clearly visible as magenta regions within the image."
- [methods] Pew2 was written in Python using the “NumPy” and “PySide2” libraries and is available on Git Hub (https://github.com/djdt/pewpew): [evidence span withheld — anchor status: not_found]
- [readme] Pewlib is a library for importing, processing and exporting LA-ICP-MS data: [evidence span withheld — anchor status: external_doc]
