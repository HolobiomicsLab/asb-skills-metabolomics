---
name: scientific-figure-export
description: Use when rendering elemental images (e.g. 66Zn, 56Fe, 55Mn) for publication
  or interpretation, where colormap choice and scale/color bar accuracy must reflect
  laser and calibration parameters.
when_to_use_negative:
- Data has not yet been imported or calibrated in Pew2 — first use the import and
  calibration workflow before figure export.
- The goal is quantitative analysis (e.g. segmentation, RGB overlay, colocalization
  statistics) rather than visual presentation; use dedicated analysis tools instead.
- Input is already a rendered figure file rather than elemental image data — this
  skill operates on the underlying NumPy/array image, not on pre-existing bitmaps.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Pew2
  role: GUI and image-processing application used to import vendor data, configure
    colormap/scale/color bars, and export publication-ready figures
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library for importing and exporting LA-ICP-MS data, used for batch
    conversion and scripting figure-generation workflows
  repo: https://github.com/djdt/pewlib
- name: Python
  role: Scripting environment for programmatic figure export and batch rendering using
    pewlib/NumPy
- name: NumPy
  role: Array backend for holding the elemental image data being rendered
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:c7d55befbd1cb1717766f1fe3c75bfa6a4f8e65c7525e79c6082135323aa6377
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
yes_rate: 0.8571428571428571
n_claims: 7
---

# scientific-figure-export

## Summary

Exports publication-ready elemental images from LA-ICP-MS data with accurate physical scaling and perceptually uniform color bars, so that visual interpretation is not distorted by colormap artifacts or wrong aspect/scale bars.

## When to use

Use when rendering elemental images (e.g. 66Zn, 56Fe, 55Mn) for publication or interpretation, where colormap choice and scale/color bar accuracy must reflect laser and calibration parameters.

## When NOT to use

- Data has not yet been imported or calibrated in Pew2 — first use the import and calibration workflow before figure export.
- The goal is quantitative analysis (e.g. segmentation, RGB overlay, colocalization statistics) rather than visual presentation; use dedicated analysis tools instead.
- Input is already a rendered figure file rather than elemental image data — this skill operates on the underlying NumPy/array image, not on pre-existing bitmaps.

## Inputs

- elemental image from LA-ICP-MS data (e.g. Agilent Mass Hunter batch, Thermo Qtegra export, Perkin Elmer Elan, text image, file-per-line export)
- laser spot size and scan speed parameters
- calibration standards (e.g. gelatin standards)
- colorbar and colormap settings

## Outputs

- publication-ready elemental image with accurate scale bar and color bar
- figure with perceptually uniform colormap rendering of the same data
- side-by-side colormap comparison image (e.g. colormap_comparison.png)

## How to apply

Render the elemental image with a perceptually uniform colormap (e.g. viridis) rather than the default rainbow (jet), since Jet's sharp contrast changes can introduce perceptual artifacts while Viridis provides uniform perceived color change. Before export, enter the laser spot size (e.g. 10 μm) and scan speed (e.g. 40 μm s−1) manually so the image aspect and scale bar are correctly sized. Ensure the color bar depicts calibration parameters (e.g. gelatin standard calibration coefficients) so that quantitative interpretation is possible. Use the built-in standardization tool on calibration standards (e.g. Agilent Mass Hunter batches) to produce quantified images before final figure export.

## Related tools

- **Pew2** (GUI and image-processing application used to import vendor data, configure colormap/scale/color bars, and export publication-ready figures) — https://github.com/djdt/pewpew
- **pewlib** (Python library for importing and exporting LA-ICP-MS data, used for batch conversion and scripting figure-generation workflows) — https://github.com/djdt/pewlib
- **Python** (Scripting environment for programmatic figure export and batch rendering using pewlib/NumPy)
- **NumPy** (Array backend for holding the elemental image data being rendered)

## Examples

```
from pewlib import load
img = load('66Zn_image.npy')
plt.imshow(img, cmap='viridis'); plt.colorbar(label='µg/g')
plt.savefig('colormap_comparison.png', dpi=300)
```

## Evaluation signals

- Scale bar reflects the manually entered laser spot size (e.g. 10 μm) and scan speed (e.g. 40 μm s−1), giving correct image aspect.
- Color bar depicts accurate calibration parameters (e.g. intercept/gradient from gelatin standard coefficients), not arbitrary units.
- Figure uses a perceptually uniform colormap (e.g. viridis) rather than jet/rainbow, so identical data appear free of artificial contrast bands.
- Exported figure matches the underlying NumPy image data — panel-to-panel comparisons of the same array show consistent values under different colormaps.
- Output file (e.g. colormap_comparison.png) exists and both panels display the same data with visibly different perceptual contrast.

## Limitations

- Rainbow colormaps (e.g. Jet) can introduce perceptual artifacts; using them risks misinterpretation of LA-ICP-MS images.
- Laser spot size and speed must be entered manually after import — automatic detection is not guaranteed, so incorrect values produce wrong scale bars and aspect.
- Colormap choice affects only visual presentation; it does not change the underlying calibrated values.
- Requires the image to have been imported and standardized first; raw vendor data cannot be exported as a calibrated figure without the calibration step.

## Evidence

- [results] Jet vs Viridis comparison: [evidence span withheld — anchor status: not_found]
