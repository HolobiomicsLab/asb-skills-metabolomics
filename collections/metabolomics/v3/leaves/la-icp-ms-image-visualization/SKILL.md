---
name: la-icp-ms-image-visualization
description: Use when visualizing 2D LA-ICP-MS elemental images (e.g., 66Zn, 55Mn,
  56Fe) for publication or interpretation and you must choose a colormap that does
  not introduce false perceptual boundaries or obscure signal structure.
when_to_use_negative:
- The image is a scalar map where a nonuniform colormap is deliberately used to highlight
  a specific threshold (e.g., segmentation masks from Otsu or k-means) — use a binary/qualitative
  palette instead.
- The data are 1D line scans or spot-wise profiles rather than a 2D raster image,
  where colormap choice is irrelevant.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Pew2
  role: Provides built-in perceptually uniform color tables for rendering LA-ICP-MS
    images and modern visualization defaults that minimize interpretation errors and
    image anomalies
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Imports and exports the calibrated elemental image data (e.g., Agilent MassHunter
    batches, text images, NumPy arrays) that feed the rendering step
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: Holds the 2D elemental image array that is passed to the plotting backend
    for colormap rendering
- name: Python
  role: Scripting environment in which the matplotlib side-by-side comparison figure
    is generated
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:d8e7a6684fcf95a7e47e1e7c73b06f254c3a31b45fb28c9badc09da75a7198a8
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
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# la-icp-ms-image-visualization

## Summary

Render LA-ICP-MS elemental images with perceptually uniform colormaps (e.g., Viridis) rather than default rainbow maps (e.g., Jet), so that displayed color transitions accurately reflect signal changes and perceptual artifacts from nonuniform contrast are avoided.

## When to use

Use when visualizing 2D LA-ICP-MS elemental images (e.g., 66Zn, 55Mn, 56Fe) for publication or interpretation and you must choose a colormap that does not introduce false perceptual boundaries or obscure signal structure.

## When NOT to use

- The image is a scalar map where a nonuniform colormap is deliberately used to highlight a specific threshold (e.g., segmentation masks from Otsu or k-means) — use a binary/qualitative palette instead.
- The data are 1D line scans or spot-wise profiles rather than a 2D raster image, where colormap choice is irrelevant.

## Inputs

- 2D elemental image array from LA-ICP-MS (e.g., precalibrated NumPy export of 66Zn from a pancreatic adenocarcinoma tissue microarray core, or raw Agilent MassHunter batch importable via pewlib)

## Outputs

- Side-by-side matplotlib figure (colormap_comparison.png) with two panels of the same image under Jet and Viridis, each with labeled colorbars and accurate scale bars

## How to apply

Load the precalibrated elemental image (e.g., the 66Zn export from pewlib or a synthetic LA-ICP-MS raster) as a 2D array and render it twice in a single matplotlib figure: once with the default rainbow colormap (jet) and once with a perceptually uniform colormap (Viridis or one of Pew2's built-in perceptually uniform color tables). Jet compresses perceived contrast unevenly across its color ramp, so equal signal steps look different in size depending on location, which can fabricate apparent edges in tissue-section imagery; Viridis maps signal to luminance monotonically so no such risk arises. Attach labeled colorbars showing calibrated concentration units to both panels, and annotate the underlying laser parameters (spot size, scan speed, acquisition time) if the image was imported via pewlib so the scale bar is accurate. Judge the result by confirming both panels display the same numeric data with visibly different perceptual contrast and that no spurious contour appears in the Viridis panel where the Jet panel suggests one.

## Related tools

- **Pew2** (Provides built-in perceptually uniform color tables for rendering LA-ICP-MS images and modern visualization defaults that minimize interpretation errors and image anomalies) — https://github.com/djdt/pewpew
- **pewlib** (Imports and exports the calibrated elemental image data (e.g., Agilent MassHunter batches, text images, NumPy arrays) that feed the rendering step) — https://github.com/djdt/pewlib
- **NumPy** (Holds the 2D elemental image array that is passed to the plotting backend for colormap rendering)
- **Python** (Scripting environment in which the matplotlib side-by-side comparison figure is generated)

## Examples

```
import matplotlib.pyplot as plt, numpy as np; img = np.load('zn66_precalibrated.npy'); fig, ax = plt.subplots(1,2); ax[0].imshow(img, cmap='jet'); ax[1].imshow(img, cmap='viridis'); fig.savefig('colormap_comparison.png')
```

## Evaluation signals

- Both panels display the identical underlying 2D array (numerically identical data, differing only in colormap).
- The Jet panel shows perceptible contrast steps or false contours that are absent in the Viridis panel.
- Colorbars on both panels show the same calibrated concentration range (e.g., the 66Zn calibration gradient of 4.12 with r² 0.9915).
- Scale bar dimensions reflect the entered laser spot size (10 µm) and scan speed (40 µm/s), confirming correct aspect.
- Output file colormap_comparison.png exists and is a single figure containing two labeled panels.

## Limitations

- The technique addresses perceptual artifacts from colormap choice only; it does not correct for instrumental drift, spikes, or background signal that require Pew2's drift tool, spike removal, or segmentation first.
- Requires the image to already be calibrated or precalibrated (e.g., via gelatin standards with reported intercept/gradient coefficients); uncalibrated raw counts may render misleadingly.
- Pew2's GUI supports drag-and-drop rendering, but scripted reproduction of the comparison requires pewlib plus matplotlib rather than a one-line CLI.

## Evidence

- [readme] README documents the pip-installable pewlib used to import and export the underlying image data that is rendered.: [evidence span withheld — anchor status: external_doc]
