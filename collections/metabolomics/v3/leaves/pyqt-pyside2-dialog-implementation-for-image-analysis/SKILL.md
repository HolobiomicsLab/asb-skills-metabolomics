---
name: pyqt-pyside2-dialog-implementation-for-image-analysis
description: Use when building a cross-platform scientific image analysis GUI in Python
  that needs interactive dialogs to compute and display image-derived statistics (e.g.
  colocalisation coefficients) on user-selected regions of imported elemental images.
when_to_use_negative:
- When the goal is scripted/batch coefficient calculation without a GUI — use the
  pewlib Python library directly instead of a PySide2 dialog.
- When input channels do not share identical pixel dimensions — colocalisation coefficient
  computation in Pew2 assumes both channels cover the same pixel grid.
- When the data are not LA-ICP-MS (or imzML) elemental images importable by Pew2/pewlib
  — this skill is grounded in vendor formats such as Agilent batches, Thermo iCap
  CSV/LDR, and PerkinElmer ELAN exports.
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Pew2
  role: The open-source LA-ICP-MS image processing application whose colocalisation
    dialog implements this skill; written in Python using NumPy and PySide2.
  repo: https://github.com/djdt/pewpew
- name: PySide2
  role: Provides the cross-platform Qt5 GUI framework used to build the interactive
    colocalisation dialog with selection-aware entry points and copyable results output.
  repo: https://github.com/djdt/pewpew
- name: NumPy
  role: Array backend on which the Li ICQ, Pearson R (and ρ shuffling), and Manders/Costes
    coefficients are computed over the selected pixel region.
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library implementing import and export of LA-ICP-MS data, feeding the
    elemental image arrays that the dialog analyses; also enables equivalent scripted
    batch processing.
  repo: https://github.com/djdt/pewlib
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:f6d9cabed9478c5c85c155ef8a98e94d743352f66e87ee9d6ba4f5596982467f
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
  edam_operation: http://edamontology.org/operation_3443
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# pyqt-pyside2-dialog-implementation-for-image-analysis

## Summary

Implement interactive analysis dialogs (e.g. colocalisation coefficient calculation) in a PySide2/Qt5 GUI backed by NumPy arrays, as done in Pew2's LA-ICP-MS image processing software.

## When to use

Use when building a cross-platform scientific image analysis GUI in Python that needs interactive dialogs to compute and display image-derived statistics (e.g. colocalisation coefficients) on user-selected regions of imported elemental images.

## When NOT to use

- When the goal is scripted/batch coefficient calculation without a GUI — use the pewlib Python library directly instead of a PySide2 dialog.
- When input channels do not share identical pixel dimensions — colocalisation coefficient computation in Pew2 assumes both channels cover the same pixel grid.
- When the data are not LA-ICP-MS (or imzML) elemental images importable by Pew2/pewlib — this skill is grounded in vendor formats such as Agilent batches, Thermo iCap CSV/LDR, and PerkinElmer ELAN exports.

## Inputs

- Two or more imported LA-ICP-MS elemental image channels as NumPy arrays with identical pixel dimensions (e.g. 55Mn and 66Zn from Agilent MassHunter batch import via pewlib)
- An optional user-defined region-of-interest from a lasso or rectangle selection that restricts coefficient computation to that region
- Optional calibration/overlay context: RGB overlays such as 55Mn (red), 56Fe (green), 66Zn (blue) where colocalised regions appear magenta

## Outputs

- Li ICQ coefficient value for the selected or full image region
- Pearson R correlation coefficient value computed from the pixel values of the two images
- Pearson R probability ρ returned after Monte Carlo comparison against random shuffles of the two channels
- Manders overlap coefficients computed using the Costes method
- A copyable results output string in the dialog displaying all computed coefficients

## How to apply

Pew2 was written in Python using the NumPy and PySide2 libraries, where PySide2 provides the cross-platform Qt5 GUI and NumPy holds the imported LA-ICP-MS elemental images (e.g. 55Mn and 66Zn channels sharing identical pixel dimensions) as arrays for computation. To replicate this skill, wire dialog entry points from both the image context menu and a lasso/rectangle selection so that a user-selected region restricts the coefficient inputs to that region only. Inside the dialog, implement the colocalisation coefficients with NumPy vectorized arithmetic over the selected pixels: the Li ICQ coefficient, Pearson's R correlation coefficient computed from the pixel values of the two images with an associated probability ρ derived by Monte Carlo comparison against random shuffles of the two channels, and Manders overlap coefficients via the Costes threshold-dependent method (doi 10.1529/biophysj.103.038422). Populate the dialog's left-side display with the calculated Li ICQ, Pearson R (with a button triggering the ρ shuffle-based calculation), and Manders values, and expose the results as a copyable output text. The rationale for this architecture is that Qt5/PySide2 dialogs coupled to NumPy let the same array backend serve both interactive GUI exploration (Pew2) and scripted batch processing (pewlib), keeping results reproducible across vendor data imports.

## Related tools

- **Pew2** (The open-source LA-ICP-MS image processing application whose colocalisation dialog implements this skill; written in Python using NumPy and PySide2.) — https://github.com/djdt/pewpew
- **PySide2** (Provides the cross-platform Qt5 GUI framework used to build the interactive colocalisation dialog with selection-aware entry points and copyable results output.) — https://github.com/djdt/pewpew
- **NumPy** (Array backend on which the Li ICQ, Pearson R (and ρ shuffling), and Manders/Costes coefficients are computed over the selected pixel region.) — https://github.com/djdt/pewpew
- **pewlib** (Python library implementing import and export of LA-ICP-MS data, feeding the elemental image arrays that the dialog analyses; also enables equivalent scripted batch processing.) — https://github.com/djdt/pewlib

## Examples

```
import numpy as np, pew; img = pew.load('sample.npy'); r = np.corrcoef(img.get('55Mn').data.ravel(), img.get('66Zn').data.ravel())[0,1]
```

## Evaluation signals

- Computed Pearson R and Li ICQ values on the Mn/Zn channel pair reproduce a positive correlative spatial relationship consistent with the reported magenta-overlay colocalisation (colocalised Mn and Zn appear as magenta regions in the RGB overlay).
- The ρ button returns a probability by shuffle-based Monte Carlo comparison, i.e. the value lies in [0,1] and changes with the number of random shuffles performed.
- Applying a lasso or rectangle selection restricts the coefficient inputs to that region only — coefficients computed on a selected subset differ from full-image values and match a manual recomputation over the selected pixels.
- Both image and selection context-menu entry points launch the same dialog, and the dialog's left-side display shows Li ICQ, Pearson R (with a ρ trigger button), and Manders values with a copyable output.
- Manders coefficients are consistent with the Costes method (doi 10.1529/biophysj.103.038422), i.e. threshold-dependent rather than unthresholded overlap.

## Limitations

- Spot-wise data import requires at least one element to be present across the entirety of the sample with similar peak positions for each element, which constrains what channel pairs can be analysed.
- Monte Carlo ρ estimation is shuffle-based, so the reported probability depends on the number of shuffles performed and is triggered manually via a dedicated button rather than computed automatically.
- The article demonstrates colocalisation on a 1.5 mm human pancreatic adenocarcinoma tissue microarray core with 55Mn/56Fe/66Zn; behaviour on other element pairs or sample types is not characterised in the source text.

## Evidence

- [methods] Pew2 was written in Python using the “NumPy” and “PySide2” libraries: [evidence span withheld — anchor status: not_found]
- [results] Pew2 quantifies the spatial overlap of two elemental images by calculating Pearson's R correlation coefficient from the pixel values of the two images: [evidence span withheld — anchor status: not_found]
- [abstract] A Python library “pewlib” has been made available: [evidence span withheld — anchor status: not_found]
