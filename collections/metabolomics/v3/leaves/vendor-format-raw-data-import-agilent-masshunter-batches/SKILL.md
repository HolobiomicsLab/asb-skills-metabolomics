---
name: vendor-format-raw-data-import-agilent-masshunter-batches
description: Use when the input is a raw LA-ICP-MS vendor export — e.g. an Agilent
  MassHunter G7200B+ .b batch directory, Perkin Elmer Elan .xl directory, or Thermo
  Qtegra/iCap CSV/LDR export — and it must be converted into image arrays without
  manual preprocessing.
when_to_use_negative:
- Data is already a preprocessed image or NumPy/CSV export — importing via the generic
  text-image importer rather than the vendor reader is unnecessary
- Data is MALDI-MSI or another non-LA-ICP-MS modality whose formats Pew2's vendor
  importers do not cover
- Only Iolite laser-log syncing is needed without the underlying vendor ICP data files
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: Pew2
  role: GUI application performing drag-and-drop import of raw vendor batches and
    applying instrument parameters to the image configuration
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library implementing the vendor-format import/export logic callable
    from scripts for batch conversion
  repo: https://github.com/djdt/pewlib
- name: pewpew
  role: command-line entry point to run Pew2 after pip installation from the cloned
    repository
  repo: https://github.com/djdt/pewpew
- name: NumPy
  role: in-memory array storage of the imported per-isotope image arrays
- name: PySide2
  role: Qt5 GUI framework backing the drag-and-drop import interface
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:11004659ac28e279b296527779e3ee3f3b47016f2e3ee4015d5632953c30199c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewpew
  tools:
  - Agilent ICP-MS
  - NumPy
  - Perkin Elmer ICP-MS
  - Pew2
  - PySide2
  - Python
  - pewlib
  - pewpew
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# vendor-format-raw-data-import-agilent-masshunter-batches

## Summary

Import raw LA-ICP-MS vendor instrument exports (Agilent MassHunter .b batches, Perkin Elmer Elan .xl directories, Thermo Qtegra/iCap CSV and LDR files) directly into image form without manual preprocessing, as implemented by Pew2/pewlib.

## When to use

Use when the input is a raw LA-ICP-MS vendor export — e.g. an Agilent MassHunter G7200B+ .b batch directory, Perkin Elmer Elan .xl directory, or Thermo Qtegra/iCap CSV/LDR export — and it must be converted into image arrays without manual preprocessing.

## When NOT to use

- Data is already a preprocessed image or NumPy/CSV export — importing via the generic text-image importer rather than the vendor reader is unnecessary
- Data is MALDI-MSI or another non-LA-ICP-MS modality whose formats Pew2's vendor importers do not cover
- Only Iolite laser-log syncing is needed without the underlying vendor ICP data files

## Inputs

- Agilent MassHunter G7200B-or-later batch (.b) directory, binary or CSV data, one sample per line
- Perkin Elmer ELAN .xl directory
- Thermo Qtegra / iCap CSV export
- Thermo iCap LDR (laser-data-reduction) export
- generic CSV image (2D delimited text) or file-per-line exports
- imzML data (via import wizard with mass selection)
- Nu Instruments Vitesse images
- Iolite laser logs (ActiveView2 or Chromium3 style) for syncing with ICP data

## Outputs

- per-isotope 2D image arrays (one per element) assembled from per-line signal arrays
- element/isotope names extracted from the acquisition metadata
- image configuration with instrument parameters (e.g. MS acquisition time) applied
- internal archive suitable for calibration, drift correction and filtering steps in Pew2

## How to apply

Detect the vendor format of the incoming export by file structure and extension: .b batch directories (Agilent, binary or CSV), .xl directories (Perkin Elmer), CSV or LDR exports (Thermo iCap), and generic CSV/line or imzML for other vendors. Parse the vendor-specific binary or delimited data to extract per-line signal arrays for each isotope, reading isotope names from the acquisition metadata; pewlib implements the actual import/export logic that both the Pew2 GUI and Python scripts call. Importing files also reads available instrument parameters such as the MS acquisition time and applies them to the image configuration, though the laser spot size and speed may need to be entered manually afterwards to ensure correct image aspect and scale bar sizing. For Agilent MassHunter data the import reads directly from the Mass Hunter binary files created on collection of each line, and calibration standards can be imported the same way before being loaded into the built-in standardization tool. If spot-wise acquisition is involved, one element must be present across the entire sample with similar peak positions across elements so a single 'marker' element can guide peak integration.

## Related tools

- **Pew2** (GUI application performing drag-and-drop import of raw vendor batches and applying instrument parameters to the image configuration) — https://github.com/djdt/pewpew
- **pewlib** (Python library implementing the vendor-format import/export logic callable from scripts for batch conversion) — https://github.com/djdt/pewlib
- **pewpew** (command-line entry point to run Pew2 after pip installation from the cloned repository) — https://github.com/djdt/pewpew
- **NumPy** (in-memory array storage of the imported per-isotope image arrays)
- **PySide2** (Qt5 GUI framework backing the drag-and-drop import interface)

## Examples

```
git clone https://github.com/djdt/pewpew; cd pewpew; pip install -r requirements.txt; pip install -e .; pewpew  # then drag the .b batch folder into the main window
```

## Evaluation signals

- Imported image preserves the per-line pixel ordering of the raw vendor export
- Instrument parameters such as MS acquisition time are read from the export and appear in the image configuration
- For Agilent MassHunter: signal for each line is read directly from the Mass Hunter binary files created on collection of each line
- Spot-wise imports require at least one element present across the entire sample with similar peak positions across elements, with a marker element guiding integration
- Image aspect and scale bar sizing are correct after the laser spot size and speed are entered manually

## Limitations

- Laser spot size and speed are not auto-imported and must be entered manually after import to get correct image aspect and scale bar
- Spot-wise import requires at least one element present across the entire sample and similar peak positions across elements; Pew2 uses a single 'marker' element to guide peak integration
- Agilent batch import is supported for MassHunter G7200B or later; earlier versions are not covered
- Peaks may be inaccurate when background signal is not uniform; peak-detection techniques (simple/windowed thresholding, continuous wavelet transforms) are applied for accurate peak detection
- Vendor support is limited to Agilent, Thermo, PerkinElmer (pewlib) plus Nu Vitesse, imzML, CSV images/lines and Iolite logs (Pew2); other formats need the generic importers

## Evidence

- [results] vendor formats listed: [evidence span withheld — anchor status: not_found]
- [readme] supported formats list: [evidence span withheld — anchor status: external_doc]
- [readme] pewlib role: [evidence span withheld — anchor status: external_doc]
