---
name: vendor-format-raw-data-import-thermo-qtegra-csv
description: Use when the input is an unconverted vendor acquisition export (Agilent
  .b batch, Perkin Elmer ELAN .xl directory, Thermo iCap/Qtegra CSV or LDR file, Nu
  Vitesse image, generic 2D delimited text image, or one-file-per-line CSV set) that
  must become a LA-ICP-MS image without manual reshaping.
when_to_use_negative:
- Data are already in an internal/precalibrated format (e.g. a precalibrated NumPy
  export) — no vendor parsing is needed and import would risk double-processing.
- The vendor export is not one of the supported formats and the data cannot be expressed
  as a 2D delimited text image or file-per-line CSV — the flexible text importers
  will not recover the acquisition structure.
- The goal is iolite-style laser-log synchronization (ActiveView2/Chromium3 logs)
  rather than plain data import — use the dedicated iolite laser-log wizard instead.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: pewlib
  role: Python library implementing the import/export of vendor LA-ICP-MS data; scriptable
    batch conversion of raw exports
  repo: https://github.com/djdt/pewlib
- name: pew2 (pewpew)
  role: GUI front end that accepts drag-and-drop import of raw Agilent, Perkin Elmer,
    Thermo, Nu, imzML, and text-image files
  repo: https://github.com/djdt/pewpew
- name: NumPy
  role: Backing array format for the imported signal data inside pewlib/pew2
- name: PySide2
  role: Qt5 GUI layer exposing the drag-and-drop import workflow
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:4e1893c62982b62802f441cd15752f7603963a1c3722c69a564bf40b060970af
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewlib
  tools:
  - NumPy
  - Perkin Elmer ICP-MS
  - Pew2
  - PySide2
  - Python
  - Thermo ICP-MS
  - pew2 (pewpew)
  - pewlib
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# vendor-format-raw-data-import-thermo-qtegra-csv

## Summary

Direct drag-and-drop import of raw LA-ICP-MS vendor exports — Agilent MassHunter .b batches (G7200B or later), Perkin Elmer ELAN 'XL' directories, Thermo iCap/Qtegra CSV and LDR exports, and generic text image / file-per-line files — into pewlib/pew2, eliminating manual preprocessing before imaging.

## When to use

Use when the input is an unconverted vendor acquisition export (Agilent .b batch, Perkin Elmer ELAN .xl directory, Thermo iCap/Qtegra CSV or LDR file, Nu Vitesse image, generic 2D delimited text image, or one-file-per-line CSV set) that must become a LA-ICP-MS image without manual reshaping.

## When NOT to use

- Data are already in an internal/precalibrated format (e.g. a precalibrated NumPy export) — no vendor parsing is needed and import would risk double-processing.
- The vendor export is not one of the supported formats and the data cannot be expressed as a 2D delimited text image or file-per-line CSV — the flexible text importers will not recover the acquisition structure.
- The goal is iolite-style laser-log synchronization (ActiveView2/Chromium3 logs) rather than plain data import — use the dedicated iolite laser-log wizard instead.

## Inputs

- Agilent MassHunter batch (.b) directory, binary or CSV data, G7200B or later
- Perkin Elmer ELAN .xl directory
- Thermo iCap Qtegra CSV export
- Thermo iCap LDR (laser-data-reduction) export
- Nu Instruments Vitesse image
- Generic 2D delimited text image (text image CSV)
- One-file-per-csv line export set
- imzML-format data via the import wizard

## Outputs

- Internal pew2/pewlib image object (NumPy-backed signal arrays per isotope)
- Imported isotope names and image configuration with instrument parameters (e.g. MS acquisition time) applied

## How to apply

Point pew2's importer (or pewlib's file-loading API) directly at the raw vendor artifact: drop the Agilent .b batch directory, Thermo iCap/Qtegra CSV/LDR export, or Perkin Elmer ELAN 'XL' directory onto the main view, or call the matching pewlib reader for delimited text images and file-per-line exports. The importer parses the vendor-specific binary or delimited payload to recover per-line signal arrays for each isotope, reading isotope names from the acquisition metadata rather than requiring a user-supplied element list. It then automatically reads any available instrument parameters — such as the MS acquisition time — and applies them to the image configuration, so the image aspect and calibration plumbing are pre-populated. Because Agilent Mass Hunter is directly supported, calibration data (e.g. gelatin standards) can be imported by the same drag-and-drop of batch folders, reading directly from the Mass Hunter binary files created on collection of each line, so standards and samples enter the pipeline identically. After import, manually verify/enter laser spot size and speed where the vendor file lacks them, since these govern the correct image aspect and scale-bar sizing downstream.

## Related tools

- **pewlib** (Python library implementing the import/export of vendor LA-ICP-MS data; scriptable batch conversion of raw exports) — https://github.com/djdt/pewlib
- **pew2 (pewpew)** (GUI front end that accepts drag-and-drop import of raw Agilent, Perkin Elmer, Thermo, Nu, imzML, and text-image files) — https://github.com/djdt/pewpew
- **NumPy** (Backing array format for the imported signal data inside pewlib/pew2)
- **PySide2** (Qt5 GUI layer exposing the drag-and-drop import workflow)

## Examples

```
pip install pewlib; import pewlib; image = pewlib.io.load('sample/Thermo_Qtegra_export.csv')
```

## Evaluation signals

- Imported image preserves the per-line pixel ordering from the raw vendor export (spot-wise or line-by-line pattern intact).
- Instrument parameters read automatically (e.g. MS acquisition time) match the values visible in the vendor file and are applied to the image configuration.
- Calibration-standard batches imported via drag-and-drop yield signal arrays readable by the built-in standardization tool, i.e. the same import path works for sample and standard.
- An elemental image (e.g. 66Zn) rendered after import shows an accurate scale bar and color bar once laser spot size and speed are confirmed, proving aspect parameters were carried through.

## Limitations

- Spot-wise peak integration requires at least one element present across the entire sample with similar peak positions for each element; otherwise peak integration cannot be guided.
- Import reads available instrument parameters, but laser spot size and speed may still need manual entry after import to ensure correct image aspect and scale-bar sizing.
- Agilent drag-and-drop import requires MassHunter G7200B or later; earlier versions are not stated as supported.
- Support for other vendors relies only on the generic text image (2D delimited text) and file-per-line importers — structure outside those patterns is not recoverable.

## Evidence

- [results] Direct import of vendor formats including Thermo: [evidence span withheld — anchor status: not_found]
- [results] Automatic reading of instrument parameters on import: [evidence span withheld — anchor status: not_found]
- [other] pewlib implements the import of supported vendor formats: [evidence span withheld — anchor status: external_doc]
- [readme] README enumerates supported formats including Thermo CSV and LDR: [evidence span withheld — anchor status: external_doc]
- [readme] README confirms drag-and-drop of Agilent batches: [evidence span withheld — anchor status: external_doc]
- [methods] pewlib repository URL: [evidence span withheld — anchor status: not_found]
