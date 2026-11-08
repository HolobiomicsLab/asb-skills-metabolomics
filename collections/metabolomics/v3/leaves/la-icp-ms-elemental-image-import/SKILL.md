---
name: la-icp-ms-elemental-image-import
description: Use when raw LA-ICP-MS data from Agilent MassHunter (.b batches), Perkin
  Elmer Elan, Thermo Qtegra, imzML, or CSV text images needs to be loaded into Pew2
  for imaging analysis without manual conversion.
when_to_use_negative:
- Data is already exported as a precalibrated, calibrated NumPy array or processed
  image — direct import of raw vendor data is intended, not re-importing processed
  outputs
- Only spot-wise (single-point) spectra without any raster/line pattern are available
  and no marker element guides peak integration — the spot-wise importer requires
  at least one element present across the entirety of the sample with similar peak
  positions
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Pew2
  role: GUI application that performs the drag-and-drop import of vendor LA-ICP-MS
    data and auto-applies instrument parameters to the image configuration
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Underlying Python library implementing import/export of Agilent, Thermo, PerkinElmer
    and delimited text data used by Pew2
  repo: https://github.com/djdt/pewlib
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:9d69fe11f58ae4e6515f15a5038db3856fa0814d92da6ebc73bcc02b70943354
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# la-icp-ms-elemental-image-import

## Summary

Import raw vendor LA-ICP-MS acquisition data (Agilent MassHunter batches, Perkin Elmer Elan XL, Thermo Qtegra exports, delimited text images, imzML) directly into Pew2 via drag-and-drop so that elemental images are configured with instrument metadata (MS acquisition time) and aspect-correct scale bars without any preprocessing.

## When to use

Use when raw LA-ICP-MS data from Agilent MassHunter (.b batches), Perkin Elmer Elan, Thermo Qtegra, imzML, or CSV text images needs to be loaded into Pew2 for imaging analysis without manual conversion.

## When NOT to use

- Data is already exported as a precalibrated, calibrated NumPy array or processed image — direct import of raw vendor data is intended, not re-importing processed outputs
- Only spot-wise (single-point) spectra without any raster/line pattern are available and no marker element guides peak integration — the spot-wise importer requires at least one element present across the entirety of the sample with similar peak positions

## Inputs

- Agilent MassHunter batch (.b directory) with binary or csv line data
- Perkin Elmer ELAN .xl directories
- Thermo iCap CSV / LDR exports
- Nu Instruments Vitesse images
- 2D delimited text images (CSV) and file-per-line exports
- imzML files (via import wizard)
- Iolite laser logs (ActiveView2 or Chromium3 style) for syncing

## Outputs

- Configured elemental image objects inside Pew2 (per isotope channel, e.g. 56Fe, 66Zn)
- Image with correct aspect ratio, scale bar, and MS acquisition time applied
- Imported calibration standard datasets ready for the standardization tool

## How to apply

Open Pew2 and drag-and-drop the vendor batch folder (e.g., an Agilent MassHunter G7200B .b directory) into the main window; Pew2 reads the MassHunter binary files created on collection of each line and constructs the elemental image. On import, Pew2 automatically reads available instrument parameters such as MS acquisition time and applies them to the image configuration, but the laser spot size and scan speed are not always carried over — enter them manually (e.g., 10 μm spot, 40 μm/s) so the image aspect ratio and scale bar sizing are correct. The same drag-and-drop workflow imports calibration standards, which are then loaded into the built-in standardization tool. For non-vendor data, use the flexible 2D delimited text importer, the file-per-line importer, or the imzML import wizard with mass selection.

## Related tools

- **Pew2** (GUI application that performs the drag-and-drop import of vendor LA-ICP-MS data and auto-applies instrument parameters to the image configuration) — https://github.com/djdt/pewpew
- **pewlib** (Underlying Python library implementing import/export of Agilent, Thermo, PerkinElmer and delimited text data used by Pew2) — https://github.com/djdt/pewlib

## Examples

```
pewpew  # then drag-and-drop the Agilent .b batch folder into the main window
```

## Evaluation signals

- After import, the elemental image (e.g., 66Zn) displays an accurate scale bar and color bar depicting the entered laser and calibration parameters
- Image aspect ratio matches the laser spot size divided by scan speed entered manually (e.g., 10 μm / 40 μm s−1)
- MS acquisition time read from the instrument file is automatically applied to the image configuration
- Calibration standard batches import identically and load into the standardization tool without conversion errors
- For spot-wise import, at least one element must be present across the entire sample with similar peak positions for each element

## Limitations

- Import does not automatically read laser spot size and scan speed for all vendors — these must be entered manually to ensure correct image aspect and scale bar sizing
- Spot-wise data import requires at least one element present across the entirety of the sample with similar peak positions for each element
- Vendor coverage is limited to Agilent (MassHunter G7200B or later), Perkin Elmer Elan XL, Thermo Qtegra/iCap, Nu Vitesse, imzML and delimited text; other vendors require the generic text importers

## Evidence
