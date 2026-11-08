---
name: numpy-array-image-rendering
description: Use when an elemental image is held as a 2D NumPy array of LA-ICP-MS
  pixel intensities and you must visualize it faithfully or audit whether a colormap
  (e.g. Jet vs Viridis) introduces perceptual contrast artifacts in the rendered image.
when_to_use_negative:
- The image is already rendered by Pew2's own visualization layer, which by design
  uses modern visualization philosophies and avoids rainbow colormaps — a separate
  Jet-vs-Viridis audit is redundant
- The input is raw instrument data (MassHunter .b batch, Thermo LDR/CSV, PerkinElmer
  ELAN .xl) rather than an imported, calibrated NumPy array — import via Pew2 or pewlib
  first
- The goal is quantitative calibration or segmentation (e.g. gelatin standard drift
  correction, k-means/Otsu masking) rather than faithful visual display of the array
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3520
tools:
- name: Pew2
  role: Source of the precalibrated NumPy export of the elemental image array and
    reference for the perceptually uniform colormap design philosophy
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library for importing/exporting LA-ICP-MS data that yields the NumPy
    arrays to be rendered
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Holds the 2D elemental image array that is passed to the rendering backend
- name: Python
  role: Runtime for the NumPy + matplotlib rendering script
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:720cdaef269701f737c4b666407d4945301a9eda7961709600bfbda24b748404
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
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# numpy-array-image-rendering

## Summary

Render LA-ICP-MS elemental image arrays (e.g. 66Zn pixel data from a Pew2 precalibrated NumPy export) with a chosen matplotlib colormap, comparing default rainbow (Jet) against perceptually uniform alternatives (Viridis) to expose colormap-induced interpretation artifacts.

## When to use

Use when an elemental image is held as a 2D NumPy array of LA-ICP-MS pixel intensities and you must visualize it faithfully or audit whether a colormap (e.g. Jet vs Viridis) introduces perceptual contrast artifacts in the rendered image.

## When NOT to use

- The image is already rendered by Pew2's own visualization layer, which by design uses modern visualization philosophies and avoids rainbow colormaps — a separate Jet-vs-Viridis audit is redundant
- The input is raw instrument data (MassHunter .b batch, Thermo LDR/CSV, PerkinElmer ELAN .xl) rather than an imported, calibrated NumPy array — import via Pew2 or pewlib first
- The goal is quantitative calibration or segmentation (e.g. gelatin standard drift correction, k-means/Otsu masking) rather than faithful visual display of the array

## Inputs

- 2D NumPy array of elemental image pixel intensities (e.g. 66Zn LA-ICP-MS data from a Pew2 precalibrated NumPy export of an Agilent MassHunter batch)
- Colormap specification (e.g. matplotlib 'jet' and 'viridis')

## Outputs

- Side-by-side matplotlib figure (e.g. colormap_comparison.png) showing the same image under Jet and Viridis with labeled colorbars
- Explicit evidence of colormap-induced perceptual contrast differences (Jet artifacts vs uniform Viridis rendering)

## How to apply

Load the elemental image as a NumPy array — either from Pew2's precalibrated NumPy export of the Agilent MassHunter batch (e.g. the 66Zn image of the 1.5 mm human pancreatic adenocarcinoma tissue microarray core) or a synthetic array with realistic LA-ICP-MS counts. Render the same array twice with matplotlib's imshow: once with the default rainbow (jet) colormap, once with a perceptually uniform colormap such as viridis. Compose the two panels side by side in a single figure, each with a labeled colorbar so the underlying intensity scale is explicit. The rationale, per Pew2's authors, is that Jet's sharp changes in contrast can introduce perceptual artifacts into LA-ICP-MS images, while perceptually uniform colormaps make perceived color change match the data change and minimize image anomalies. Pew2 itself 'adheres to modern visualization philosophies' for this reason, and any standalone Python script using NumPy/matplotlib can reproduce the same side-by-side audit outside the GUI.

## Related tools

- **Pew2** (Source of the precalibrated NumPy export of the elemental image array and reference for the perceptually uniform colormap design philosophy) — https://github.com/djdt/pewpew
- **pewlib** (Python library for importing/exporting LA-ICP-MS data that yields the NumPy arrays to be rendered) — https://github.com/djdt/pewlib
- **NumPy** (Holds the 2D elemental image array that is passed to the rendering backend)
- **Python** (Runtime for the NumPy + matplotlib rendering script)

## Examples

```
import numpy as np, matplotlib.pyplot as plt; img = np.load('66Zn.npy'); fig, ax = plt.subplots(1, 2, figsize=(10, 5)); ax[0].imshow(img, cmap='jet'); ax[0].set_title('Jet'); ax[0].colorbar(); ax[1].imshow(img, cmap='viridis'); ax[1].set_title('Viridis'); ax[1].colorbar(); plt.savefig('colormap_comparison.png')
```

## Evaluation signals

- Both panels display the identical underlying data array — only the colormap differs, so pixel values and spatial structure must match exactly
- The Jet panel shows visible false contrast bands or edge enhancements that are absent in the Viridis panel, confirming the perceptual-artifact claim
- The output figure (e.g. colormap_comparison.png) exists and each panel has its own labeled colorbar reflecting the same intensity scale
- The colorbar increments are perceptually uniform under Viridis (equal perceived color change per unit concentration), matching Pew2's 'modern visualization philosophies' claim
- No data reinterpretation is needed: the same image reads as the same feature structure under both colormaps, differing only in perceptual salience

## Limitations

- The comparison is a visual audit, not a quantitative colorimetric analysis — it demonstrates but does not measure perceptual uniformity
- Perceptual artifacts depend on the data range and spatial gradient of the specific elemental image; Jet artifacts may be invisible in smooth, low-contrast images
- The skill assumes the array is already calibrated (Pew2 precalibrated NumPy export or equivalent); rendering uncalibrated raw counts can mislead about concentration scale
- Pew2's GUI is Windows-first via executable releases; running the pipeline elsewhere requires cloning the repo and installing via pip with requirements.txt

## Evidence

- [supplementary] Jet's sharp changes in contrast can introduce perceptual artifacts: [evidence span withheld — anchor status: not_found]
- [methods] Pew2 was written in Python using the NumPy and PySide2 libraries: [evidence span withheld — anchor status: not_found]
