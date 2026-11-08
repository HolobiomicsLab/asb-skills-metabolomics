---
name: laser-parameter-manual-entry-after-import
description: Use when raw vendor data (Agilent MassHunter batches, Perkin Elmer Elan,
  Thermo Qtegra exports) has been imported into Pew2 and the image scale bar or aspect
  ratio is incorrect because the laser spot size and speed were not auto-populated.
when_to_use_negative:
- Input is already a calibrated or pre-processed image (e.g. a NumPy export) where
  aspect and scale are already correct.
- The acquisition was spot-wise (single-point) with no scan speed, so no aspect correction
  is needed.
- Laser parameters are automatically read by the importer for the specific vendor
  format, making manual entry redundant.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Pew2
  role: GUI into which vendor data is imported and where the laser spot size and speed
    are entered manually after import
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library implementing the import of Agilent, Thermo, and PerkinElmer
    data that Pew2 relies on
  repo: https://github.com/djdt/pewlib
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:29549424c909e7d1e1a3c41c0483d109226821ca5f88d2f2bc9bbb7741c5b739
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - Pew2
  - Python
  - pewlib
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# laser-parameter-manual-entry-after-import

## Summary

After importing LA-ICP-MS data into Pew2, manually enter the laser spot size and scan speed so the image aspect ratio and scale bar reflect the true acquisition geometry. This skill corrects the automated import, which reads instrument parameters such as MS acquisition time but not the physical laser dimensions.

## When to use

Use when raw vendor data (Agilent MassHunter batches, Perkin Elmer Elan, Thermo Qtegra exports) has been imported into Pew2 and the image scale bar or aspect ratio is incorrect because the laser spot size and speed were not auto-populated.

## When NOT to use

- Input is already a calibrated or pre-processed image (e.g. a NumPy export) where aspect and scale are already correct.
- The acquisition was spot-wise (single-point) with no scan speed, so no aspect correction is needed.
- Laser parameters are automatically read by the importer for the specific vendor format, making manual entry redundant.

## Inputs

- raw Agilent MassHunter batch folder (.b directory, G7200B or later)
- Perkin Elmer Elan “XL” directory
- Thermo Qtegra export
- text image (2D delimited) or file-per-line export
- known laser spot size (μm) and scan speed (μm/s) from the acquisition protocol

## Outputs

- Pew2 image object with correct pixel aspect ratio
- accurately sized scale bar reflecting the true laser spot size and scan speed

## How to apply

Drag-and-drop the raw Agilent batch (MassHunter G7200B or later), Perkin Elmer Elan “XL”, or Thermo Qtegra export into Pew2; the importer automatically reads instrument parameters such as MS acquisition time and applies them to the image configuration. Immediately after import, enter the laser spot size and scan speed manually — for example, a 10 μm beam diameter with a 40 μm/s scan speed used for the pancreatic adenocarcinoma tissue microarray acquisition. Rationale: the correct spot-to-speed ratio determines pixel aspect; with a 0.25 s acquisition time the spot speed defines the pixel width, so omitting these values distorts the scale bar and the apparent tissue geometry. Check that the image aspect and scale bar sizing match the physical scan dimensions before proceeding to segmentation, quantification, or co-registration steps.

## Related tools

- **Pew2** (GUI into which vendor data is imported and where the laser spot size and speed are entered manually after import) — https://github.com/djdt/pewpew
- **pewlib** (Python library implementing the import of Agilent, Thermo, and PerkinElmer data that Pew2 relies on) — https://github.com/djdt/pewlib

## Evaluation signals

- The image aspect ratio (width-to-height in pixels) matches the ratio of total scan length to number of lines implied by spot size and line spacing.
- The scale bar in the rendered Pew2 image shows the physical length expected for the acquisition (e.g. 100 μm for a 10 μm spot, 40 μm/s scan).
- Visual check: tissue structures (e.g. the 1.5 mm microarray core) appear geometrically round/undistorted, not elongated or compressed.
- The manually entered spot size and speed persist in the image configuration and are reflected downstream when quantification or segmentation is applied.

## Limitations

- The importer does not automatically extract laser spot size or scan speed from all vendor formats, so the values must be known and entered by hand.
- If the laser parameters are entered incorrectly, the image geometry and scale bar will be wrong and any downstream quantification or segmentation will inherit the distortion.
- Only applies to line-scan imaging acquisitions; spot-wise or ablation-time-aligned data may not have an analogous speed parameter.

## Evidence

- [methods] Pew2 was written in Python using the “NumPy” and “PySide2” libraries and is available on Git Hub: [evidence span withheld — anchor status: not_found]
