---
name: perceptually-uniform-colormap-selection
description: Use when rendering LA-ICP-MS elemental images (e.g., 66Zn, 55Mn, 56Fe
  ion images) for publication or presentation, or when auditing existing figures for
  colormap-induced perceptual artifacts.
when_to_use_negative:
- When the image is a segmentation mask, binary ROI mask, or RGB overlay (e.g., the
  55Mn/56Fe/66Zn magenta-colocalization overlay), where discrete color assignment
  rather than continuous perceptual uniformity governs legibility
- When the target output is a non-visual quantitative product (e.g., calibration coefficients,
  drift-corrected signal tables) rather than a rendered image for human interpretation
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3077
- http://edamontology.org/topic_3315
tools:
- name: Pew2
  role: GUI for importing LA-ICP-MS data and applying visualization philosophies that
    minimize image anomalies; supports the perceptually uniform colormap convention
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library for importing/exporting LA-ICP-MS data used to load elemental
    images before colormap rendering
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Provides the 2D image array (elemental image data) that both colormaps encode
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:c873c6f10749e1d19d1bb35473e37c33220ebc52bd523ea4611b134a4c2d2939
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - Python
  - pewlib
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3077
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# perceptually-uniform-colormap-selection

## Summary

Selecting perceptually uniform colormaps (e.g., viridis) over default rainbow colormaps (e.g., jet) when rendering LA-ICP-MS elemental images, because rainbow colormaps introduce sharp changes in perceived contrast that can create visual artifacts and data-interpretation errors.

## When to use

Use when rendering LA-ICP-MS elemental images (e.g., 66Zn, 55Mn, 56Fe ion images) for publication or presentation, or when auditing existing figures for colormap-induced perceptual artifacts.

## When NOT to use

- When the image is a segmentation mask, binary ROI mask, or RGB overlay (e.g., the 55Mn/56Fe/66Zn magenta-colocalization overlay), where discrete color assignment rather than continuous perceptual uniformity governs legibility
- When the target output is a non-visual quantitative product (e.g., calibration coefficients, drift-corrected signal tables) rather than a rendered image for human interpretation

## Inputs

- LA-ICP-MS elemental image (e.g., 66Zn) as a 2D NumPy array exported from pewlib or loaded from a precalibrated NumPy export
- Raw instrument data (Agilent MassHunter batch, Thermo Qtegra, Perkin Elmer Elan .xl) imported via Pew2
- Calibration parameters (e.g., 66Zn intercept 8.48, gradient 4.12) for annotating accurate colorbars

## Outputs

- Side-by-side matplotlib figure (e.g., colormap_comparison.png) with two panels of the same data rendered under jet and viridis, each with labeled colorbars
- Rationale for adopting perceptually uniform colormaps in downstream figures and reports

## How to apply

Render the same elemental image (e.g., the 66Zn image imported from the Agilent MassHunter batch or the precalibrated NumPy export) twice within a single matplotlib figure: once with the default rainbow colormap (jet) and once with a perceptually uniform colormap such as viridis. Rationale: jet's nonuniform perceptual contrast introduces sharp changes that can be mistaken for real chemical boundaries, whereas viridis provides uniform perceived color change with no such risk, aligning with Pew2's design goal of minimizing data interpretation errors and image anomalies. Attach accurate colorbars depicting laser and calibration parameters, and label both panels so the comparison is explicit. Judge correctness by confirming both panels encode identical underlying pixel values and that the jet panel visibly exaggerates or distorts edges relative to the viridis panel.

## Related tools

- **Pew2** (GUI for importing LA-ICP-MS data and applying visualization philosophies that minimize image anomalies; supports the perceptually uniform colormap convention) — https://github.com/djdt/pewpew
- **pewlib** (Python library for importing/exporting LA-ICP-MS data used to load elemental images before colormap rendering) — https://github.com/djdt/pewlib
- **NumPy** (Provides the 2D image array (elemental image data) that both colormaps encode)

## Examples

```
import numpy as np, matplotlib.pyplot as plt; img = np.load('66Zn.npy'); fig, ax = plt.subplots(1,2); ax[0].imshow(img, cmap='jet'); ax[1].imshow(img, cmap='viridis'); fig.savefig('colormap_comparison.png')
```

## Evaluation signals

- Both panels in the side-by-side figure encode the same underlying pixel values (identical data, different colormaps)
- The jet panel exhibits sharp changes in perceived contrast that the viridis panel does not
- The perceptually uniform colormap (e.g., viridis) shows uniform perceived color change across the data range
- Colorbars accurately depict laser and calibration parameters as in the 66Zn figure
- Output file (e.g., colormap_comparison.png) exists and is legible when rendered

## Limitations

- Rainbow (jet) colormaps remain common defaults in some tools and can slip into figures without deliberate selection
- Perceptual uniformity addresses perceived contrast artifacts but does not correct underlying signal spikes, drift, or calibration errors that must be handled upstream
- The demonstration is qualitative (side-by-side visual comparison); no quantitative perceptual-difference metric is provided in the paper

## Evidence
