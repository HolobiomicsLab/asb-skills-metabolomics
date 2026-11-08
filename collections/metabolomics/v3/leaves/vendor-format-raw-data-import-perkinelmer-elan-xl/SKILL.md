---
name: vendor-format-raw-data-import-perkinelmer-elan-xl
description: Use when incoming LA-ICP-MS acquisition data is a raw vendor export,
  specifically a Perkin Elmer ELAN '.xl' directory (also Agilent .b batches or Thermo
  CSV/LDR), and the goal is to load it as an image without manual file conversion.
when_to_use_negative:
- Input is already a processed 2D delimited text image or NumPy archive that Pew2
  loads natively — no vendor parsing needed
- Data is an imzML mass spectrometry imaging file requiring mass selection; use the
  imzML import wizard instead
- Vendor export is an Iolite laser log to be synced with ICP data rather than directly
  imported
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Pew2
  role: GUI application that provides drag-and-drop import of Perkin Elmer ELAN '.xl'
    directories and applies vendor instrument parameters to the image configuration
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library implementing the import of PerkinElmer (and other vendor) exports
    for scripted batch conversion and processing
  repo: https://github.com/djdt/pewlib
- name: NumPy
  role: NumPy archives serve as the internal storage format produced by the vendor
    import
- name: PySide2
  role: Qt5 GUI framework in which the drag-and-drop import interface of Pew2 is implemented
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:67f3c72e2413e58ab1ae0e9e49d0743d1fe2474e3e9b5b54e3b2a0df20b82165
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - NumPy
  - Perkin Elmer ICP-MS
  - Pew2
  - PySide2
  - Python
  - pewlib
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# vendor-format-raw-data-import-perkinelmer-elan-xl

## Summary

Pew2 and its underlying pewlib library directly import raw Perkin Elmer ELAN '.xl' directory exports (alongside Agilent MassHunter and Thermo Qtegra/iCap files) into LA-ICP-MS images without any preprocessing, removing user workload at the first step of the imaging pipeline.

## When to use

Use when incoming LA-ICP-MS acquisition data is a raw vendor export, specifically a Perkin Elmer ELAN '.xl' directory (also Agilent .b batches or Thermo CSV/LDR), and the goal is to load it as an image without manual file conversion.

## When NOT to use

- Input is already a processed 2D delimited text image or NumPy archive that Pew2 loads natively — no vendor parsing needed
- Data is an imzML mass spectrometry imaging file requiring mass selection; use the imzML import wizard instead
- Vendor export is an Iolite laser log to be synced with ICP data rather than directly imported

## Inputs

- Perkin Elmer ELAN '.xl' directory (raw vendor export)
- Optionally Agilent MassHunter G7200B-or-later .b batch directories, Thermo iCap CSV/LDR exports, Nu Instruments Vitesse images, or generic CSV images/lines
- Laser spot size and scan speed values for post-import manual entry

## Outputs

- LA-ICP-MS image with per-isotope signal arrays and isotope names from acquisition metadata
- Image configuration with MS acquisition time applied from the vendor export
- NumPy archive format file usable in Pew2/pewlib workflows

## How to apply

Provide the Perkin Elmer ELAN '.xl' directory to Pew2 via its drag-and-drop import (or the pewlib import functions for scripted batches), which parses the vendor export and reads isotope names directly from the acquisition metadata. During import, any available instrument parameters such as the MS acquisition time are read and applied to the image configuration, so acquisition-time-dependent image geometry is set automatically. For spot-wise data, one element must be present across the entire sample so a single 'marker' element can guide peak integration; peak detection may use simple or windowed thresholding and continuous wavelet transforms to handle non-uniform background. After import, manually enter the laser spot size and scan speed to ensure the correct image aspect and scale bar sizing, since these are not recovered from the ELAN export. For batch conversions or pipelines outside the GUI, call the corresponding import routine in pewlib and assemble the result into the internal NumPy archive format used by Pew2.

## Related tools

- **Pew2** (GUI application that provides drag-and-drop import of Perkin Elmer ELAN '.xl' directories and applies vendor instrument parameters to the image configuration) — https://github.com/djdt/pewpew
- **pewlib** (Python library implementing the import of PerkinElmer (and other vendor) exports for scripted batch conversion and processing) — https://github.com/djdt/pewlib
- **NumPy** (NumPy archives serve as the internal storage format produced by the vendor import)
- **PySide2** (Qt5 GUI framework in which the drag-and-drop import interface of Pew2 is implemented)

## Examples

```
pip install pewlib; python -c "import pewlib; pewlib.io.reader.read_pew('sample.npy')"
```

## Evaluation signals

- Imported image preserves the per-line pixel ordering from the raw ELAN export
- MS acquisition time from the vendor export is correctly applied to the image configuration
- Isotope names appear exactly as recorded in the ELAN acquisition metadata
- Laser spot size and speed entered manually produce a correct image aspect and accurate scale bar sizing (e.g. scale bar values match the physical 40 μm/s scan speed)
- Same file imported via pewlib produces the same NumPy archive contents as via drag-and-drop in the GUI

## Limitations

- Instrument parameters such as laser spot size and scan speed are not automatically recovered — they must be entered manually after import to get correct aspect and scale bar
- Spot-wise import requires at least one element present across the entire sample with similar peak positions for each element so a single marker element can guide peak integration
- Support is limited to the listed vendor formats (Perkin Elmer ELAN '.xl', Agilent batches, Thermo CSV/LDR, Nu Vitesse, imzML); other vendors must fall back to the flexible text image or file-per-line importers

## Evidence

- [abstract] Support for other vendors is maintained with a flexible text image (2D delimited text files) and file-per-line importer.: [evidence span withheld — anchor status: not_found]
- [results] Drag-and-drop importation of raw Agilent batches (MassHunter G7200B or later), Perkin Elmer Elan “XL”, and Thermo Fischer Qtegra exports, as well as both text image and file-per-line exports: [evidence span withheld — anchor status: not_found]
- [readme] Pewlib is a library for importing, processing and exporting LA-ICP-MS data. Currently exports from Agilent, Thermo and PerkinElmer software is supported, as well as delimited text images.: [evidence span withheld — anchor status: external_doc]
