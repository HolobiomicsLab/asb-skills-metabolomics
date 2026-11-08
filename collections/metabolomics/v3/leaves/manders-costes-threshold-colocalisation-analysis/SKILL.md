---
name: manders-costes-threshold-colocalisation-analysis
description: Decision support for quantifying spatial correlation between two LA-ICP-MS
  elemental images (e.g. 55Mn vs 66Zn) when reporting colocalisation coefficients
  such as Manders, Li ICQ, or Pearson R.
when_to_use_negative:
- The input is a single-channel image or the two channels have mismatched pixel dimensions
  — coefficients require pixelwise comparison of two aligned channels
- You only need visual colocalisation inspection, e.g. an RGB overlay (55Mn red, 56Fe
  green, 66Zn blue) without quantitative coefficients
- The task is segmentation or ROI isolation rather than overlap quantification — use
  Pew2's thresholding/k-means tools instead
edam_operation: http://edamontology.org/operation_3443
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: Pew2
  role: GUI hosting the colocalisation dialog that computes Manders (Costes), Li ICQ,
    and Pearson R over the two elemental image channels
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Library that implements import of vendor data into the NumPy-array backend
    feeding the colocalisation coefficients
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Backend for vectorized pixel-value arithmetic used in the Li ICQ and Pearson
    R implementations
- name: PySide2
  role: Qt5 GUI framework for the colocalisation dialog with copyable results output
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:4a8abfcc2e6b86e5b13eb51b008c5a24519834a6a7dfe987b37dffd1eda39128
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
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# manders-costes-threshold-colocalisation-analysis

## Summary

Threshold-dependent colocalisation analysis implemented in Pew2 that computes Manders overlap coefficients via the Costes method (doi 10.1529/biophysj.103.038422) to quantify the spatial relationship between two LA-ICP-MS elemental image channels, alongside Li ICQ and Pearson R (with shuffle-based ρ probability).

## When to use

Decision support for quantifying spatial correlation between two LA-ICP-MS elemental images (e.g. 55Mn vs 66Zn) when reporting colocalisation coefficients such as Manders, Li ICQ, or Pearson R.

## When NOT to use

- The input is a single-channel image or the two channels have mismatched pixel dimensions — coefficients require pixelwise comparison of two aligned channels
- You only need visual colocalisation inspection, e.g. an RGB overlay (55Mn red, 56Fe green, 66Zn blue) without quantitative coefficients
- The task is segmentation or ROI isolation rather than overlap quantification — use Pew2's thresholding/k-means tools instead

## Inputs

- Two aligned LA-ICP-MS elemental image channels (NumPy arrays, identical pixel dimensions, e.g. 55Mn and 66Zn from a pewlib-imported sample)
- Optional lasso/rectangle selection ROI restricting coefficient computation to a sub-region

## Outputs

- Manders overlap coefficients (Costes method) for the two channels
- Li ICQ coefficient over the full image (or selection)
- Pearson R correlation coefficient
- Pearson R probability ρ (Monte Carlo shuffle-based, triggered via button)

## How to apply

Load the two LA-ICP-MS elemental image channels (e.g. 55Mn and 66Zn) into Pew2's NumPy-array backend via pewlib so they share identical pixel dimensions. Open the colocalisation dialog and compute Manders overlap coefficients using the Costes method (doi 10.1529/biophysj.103.038422), which determines the threshold below which signal is deemed background before calculating overlap. Also compute the Li ICQ coefficient over the entire image using NumPy vectorized arithmetic, and Pearson R with its associated probability ρ, obtained by comparing the R value against random shuffles of the two channels via the dialog's ρ button. Optionally use a lasso or rectangle selection so the coefficient inputs are restricted to that region only; results are displayed on the dialog's left side with a copyable output. Validate by confirming the Mn/Zn values reproduce the positive correlative relationship visible as magenta regions in the RGB overlay (Figure 6).

## Related tools

- **Pew2** (GUI hosting the colocalisation dialog that computes Manders (Costes), Li ICQ, and Pearson R over the two elemental image channels) — https://github.com/djdt/pewpew
- **pewlib** (Library that implements import of vendor data into the NumPy-array backend feeding the colocalisation coefficients) — https://github.com/djdt/pewlib
- **NumPy** (Backend for vectorized pixel-value arithmetic used in the Li ICQ and Pearson R implementations)
- **PySide2** (Qt5 GUI framework for the colocalisation dialog with copyable results output)

## Examples

```
pewpew  # open Pew2 GUI, import Agilent MassHunter batch via drag-and-drop, then open the colocalisation dialog on 55Mn and 66Zn channels
```

## Evaluation signals

- Pearson R and Li ICQ values for the 55Mn/66Zn pair are positive, consistent with the magenta colocalisation regions seen in the RGB overlay (Figure 6)
- The ρ button returns a probability in [0,1] derived from Monte Carlo comparison against random shuffles of the two channels
- Manders coefficients change when a lasso/rectangle selection is applied, confirming inputs are restricted to the selected region
- Results match Table 3 values reported for the pancreatic adenocarcinoma tissue microarray core

## Limitations

- Spot-wise data import requires at least one element present across the entire sample with similar peak positions for each element — mis-aligned channels will corrupt coefficient computation
- Pearson R probability ρ is computed only on explicit button press (shuffle-based Monte Carlo), not automatically with the coefficient
- Colocalisation coefficients depend on prior calibration and drift/spike preprocessing; unnormalized images can bias the overlap values

## Evidence

- [results] Pew2 quantifies the spatial overlap of two elemental images by calculating Pearson's R correlation coefficient from the pixel values of the two images: [evidence span withheld — anchor status: not_found]
- [methods] Pew2 supports data import and processing via NumPy/PySide2: [evidence span withheld — anchor status: not_found]
- [readme] pewlib provides import/export of LA-ICP-MS data: [evidence span withheld — anchor status: external_doc]
