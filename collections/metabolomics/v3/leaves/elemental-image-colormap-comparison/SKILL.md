---
name: elemental-image-colormap-comparison
description: Use when preparing elemental images from LA-ICP-MS data for publication
  or review and you need to show or justify why the default rainbow (Jet) colormap
  risks misinterpretation versus a perceptually uniform alternative.
when_to_use_negative:
- The image already uses a perceptually uniform colormap and no colormap justification
  is needed
- The goal is quantitative pixel calibration or drift/spike correction rather than
  visualization choice
- The input is line/spot-wise raw instrument data that has not yet been imported and
  calibrated into a 2D image
tools:
- name: Pew2
  role: source of the LA-ICP-MS images and the visualization philosophy that motivates
    the comparison; its GUI exports the calibrated image arrays used here
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library for importing and exporting the LA-ICP-MS data so the elemental
    image can be loaded as a NumPy array for rendering
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: holds the elemental image as a 2D array that is passed to matplotlib for rendering
    under each colormap
- name: Python
  role: runtime for the matplotlib rendering script that produces the side-by-side
    comparison figure
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:8689fd5df2be6ed1e02820d49becabbc054e4a3ea573ddb871d5e364d1dd3e3f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Pew2
  - Python
  - pewlib
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# elemental-image-colormap-comparison

## Summary

Renders the same LA-ICP-MS elemental image (e.g. a calibrated 66Zn tissue image) side by side under a default rainbow colormap (Jet) and a perceptually uniform colormap (Viridis) to demonstrate how nonuniform perceptual contrast in rainbow colormaps can introduce interpretation artifacts that uniform colormaps avoid.

## When to use

Use when preparing elemental images from LA-ICP-MS data for publication or review and you need to show or justify why the default rainbow (Jet) colormap risks misinterpretation versus a perceptually uniform alternative.

## When NOT to use

- The image already uses a perceptually uniform colormap and no colormap justification is needed
- The goal is quantitative pixel calibration or drift/spike correction rather than visualization choice
- The input is line/spot-wise raw instrument data that has not yet been imported and calibrated into a 2D image

## Inputs

- calibrated 2D elemental image array (e.g. 66Zn from LA-ICP-MS, precalibrated NumPy export)
- optional synthetic elemental image with realistic LA-ICP-MS pixel values

## Outputs

- side-by-side comparison figure (e.g. colormap_comparison.png) with one jet panel and one viridis panel of the same data
- labeled colorbars for each panel

## How to apply

Load a single calibrated elemental image — for example the 66Zn export from the Pew2 workflow (precalibrated NumPy export accompanying the paper) — into Python using NumPy and matplotlib. Render the identical 2D array twice: once with the default rainbow colormap (jet) and once with a perceptually uniform colormap such as viridis, adding labeled colorbars to each panel. Arrange the two panels side by side in a single matplotlib figure so that differences in perceived contrast are directly visible. The rationale, grounded in the Pew2 paper, is that Jet's sharp changes in contrast can introduce perceptual artifacts (image anomalies) while Viridis provides uniform perceived color change, minimizing data interpretation errors as part of Pew2's modern visualization philosophy.

## Related tools

- **Pew2** (source of the LA-ICP-MS images and the visualization philosophy that motivates the comparison; its GUI exports the calibrated image arrays used here) — https://github.com/djdt/pewpew
- **pewlib** (Python library for importing and exporting the LA-ICP-MS data so the elemental image can be loaded as a NumPy array for rendering) — https://github.com/djdt/pewlib
- **NumPy** (holds the elemental image as a 2D array that is passed to matplotlib for rendering under each colormap)
- **Python** (runtime for the matplotlib rendering script that produces the side-by-side comparison figure)

## Examples

```
import numpy as np, matplotlib.pyplot as plt; img = np.load('66Zn.npy'); fig, ax = plt.subplots(1, 2); ax[0].imshow(img, cmap='jet'); ax[1].imshow(img, cmap='viridis'); plt.savefig('colormap_comparison.png')
```

## Evaluation signals

- The output file (e.g. colormap_comparison.png) exists on disk
- Both panels display the same underlying 66Zn data array — only the colormap differs
- Jet panel shows visibly nonuniform perceptual contrast (e.g. apparent boundaries that do not correspond to data changes) while the Viridis panel does not
- Each panel has a labeled colorbar reflecting the same intensity scale
- The figure preserves the correct image aspect (e.g. from laser spot size and scan speed entered during import)

## Limitations

- Rainbow colormaps only risk artifacts when the underlying image has sharp intensity gradients; on featureless images the comparison may be uninformative
- The comparison is qualitative/visual — it does not quantify colocalization or concentration values

## Evidence

- [results] Jet's sharp changes in contrast can introduce perceptual artifacts, while Viridis provides uniform perceived color change with no such risk: [evidence span withheld — anchor status: not_found]
- [methods] Pew2 was written in Python using the NumPy and PySide2 libraries and is available on Git Hub: [evidence span withheld — anchor status: not_found]
- [readme] Pewlib is a library for importing, processing and exporting LA-ICP-MS data: [evidence span withheld — anchor status: external_doc]
