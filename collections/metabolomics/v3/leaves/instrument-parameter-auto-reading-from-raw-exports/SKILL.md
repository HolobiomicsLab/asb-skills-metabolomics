---
name: instrument-parameter-auto-reading-from-raw-exports
description: Use when raw vendor LA-ICP-MS exports (Agilent .b batch directories,
  Perkin Elmer Elan 'XL', Thermo Qtegra exports, or text image/file-per-line files)
  are imported into Pew2 and instrument parameters such as MS acquisition time must
  be applied to the image configuration without manual.
when_to_use_negative:
- The input is already a processed or precalibrated image (e.g. a NumPy export) that
  carries no vendor instrument metadata to read
- Parameters such as laser spot size and scan speed are absent from the export and
  must be entered manually regardless — the skill only covers parameters available
  in the vendor files
- The data format is unsupported (e.g. non-vendor formats outside Agilent, Perkin
  Elmer, Thermo, Nu Instruments Vitesse, imzML, or generic CSV images/lines)
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: Pew2
  role: GUI application whose drag-and-drop importer reads instrument parameters from
    raw vendor exports and applies them to the image configuration
  repo: https://github.com/djdt/pewpew
- name: pewlib
  role: Python library implementing import and export of LA-ICP-MS data from Agilent,
    Thermo, PerkinElmer software, underlying the parameter-reading import
  repo: https://github.com/djdt/pewlib
- name: PySide2
  role: Qt5 GUI framework used to build Pew2's drag-and-drop import interface
- name: NumPy
  role: Python numerical library used to assemble per-isotope image arrays after parameter
    reading
- name: Agilent ICP-MS
  role: Vendor whose MassHunter batch export format provides binary/csv data from
    which parameters are read
- name: Perkin Elmer ICP-MS
  role: Vendor whose Elan 'XL' directory format provides data from which parameters
    are read
- name: Thermo ICP-MS
  role: Vendor whose Qtegra export format provides data from which parameters are
    read
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:4ef411f0cd7d558bae8cd53f0c6754dc8f1fab8a663152d1e644a5a083e386a1
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
  - Thermo ICP-MS
  - pewlib
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# instrument-parameter-auto-reading-from-raw-exports

## Summary

Automatically reads available instrument parameters (e.g. MS acquisition time) directly from raw LA-ICP-MS vendor exports (Agilent MassHunter batches, Perkin Elmer Elan 'XL', Thermo Fischer Qtegra) during drag-and-drop import in Pew2 and applies them to the image configuration, removing manual parameter entry.

## When to use

Use when raw vendor LA-ICP-MS exports (Agilent .b batch directories, Perkin Elmer Elan 'XL', Thermo Qtegra exports, or text image/file-per-line files) are imported into Pew2 and instrument parameters such as MS acquisition time must be applied to the image configuration without manual preprocessing or manual parameter entry.

## When NOT to use

- The input is already a processed or precalibrated image (e.g. a NumPy export) that carries no vendor instrument metadata to read
- Parameters such as laser spot size and scan speed are absent from the export and must be entered manually regardless — the skill only covers parameters available in the vendor files
- The data format is unsupported (e.g. non-vendor formats outside Agilent, Perkin Elmer, Thermo, Nu Instruments Vitesse, imzML, or generic CSV images/lines)

## Inputs

- Agilent MassHunter batch directory (.b, G7200B or later), one sample per line, binary or csv data
- Perkin Elmer Elan 'XL' directory
- Thermo Fischer Qtegra export
- Text image (2D delimited text file) or file-per-line export

## Outputs

- Imported Pew2 image with instrument parameters (e.g. MS acquisition time) applied to the image configuration
- Per-isotope signal arrays assembled into image arrays carrying the read acquisition metadata

## How to apply

When a raw vendor export is imported by drag-and-drop into Pew2, the importer parses the vendor-specific file structure — Agilent MassHunter G7200B-or-later batch directories (with one sample per line, reading both binary and csv data), Perkin Elmer Elan 'XL' directories, or Thermo Fischer Qtegra exports — and extracts any available instrument parameters such as the MS acquisition time directly from the file metadata. The read parameters are then applied automatically to the image configuration, so that the assembled per-isotope image arrays carry the correct acquisition timing without user intervention. Any parameters not present in the export (e.g. laser spot size and scan speed) are still entered manually afterwards to ensure the correct image aspect and scale bar sizing. The rationale is that direct binary/CSV parsing of the vendor files removes any need for preprocessing and greatly reduces user workload, while avoiding transcription errors in parameters that drive image aspect and downstream calibration.

## Related tools

- **Pew2** (GUI application whose drag-and-drop importer reads instrument parameters from raw vendor exports and applies them to the image configuration) — https://github.com/djdt/pewpew
- **pewlib** (Python library implementing import and export of LA-ICP-MS data from Agilent, Thermo, PerkinElmer software, underlying the parameter-reading import) — https://github.com/djdt/pewlib
- **PySide2** (Qt5 GUI framework used to build Pew2's drag-and-drop import interface)
- **NumPy** (Python numerical library used to assemble per-isotope image arrays after parameter reading)
- **Agilent ICP-MS** (Vendor whose MassHunter batch export format provides binary/csv data from which parameters are read)
- **Perkin Elmer ICP-MS** (Vendor whose Elan 'XL' directory format provides data from which parameters are read)
- **Thermo ICP-MS** (Vendor whose Qtegra export format provides data from which parameters are read)

## Examples

```
pewpew  # then drag-and-drop the Agilent .b batch directory (or Perkin Elmer 'XL' / Thermo Qtegra export) into the main window; instrument parameters such as MS acquisition time are read and applied automatically
```

## Evaluation signals

- Confirm that the imported Pew2 image configuration shows the MS acquisition time read from the vendor export (e.g. 0.25 s in the pancreatic adenocarcinoma example) without manual entry
- Check that the assembled image preserves the per-line pixel ordering from the raw export
- Verify the image aspect and scale bar sizing are consistent once the remaining laser parameters (spot size, speed) are entered manually
- Compare the imported per-isotope arrays against a precalibrated NumPy export or raw Mass Hunter data to confirm no parameter transcription drift
- Confirm the import works for each supported vendor format (Agilent .b, Perkin Elmer 'XL', Thermo Qtegra) and for text image / file-per-line exports

## Limitations

- Only parameters present in the vendor export are read automatically; laser spot size and speed must still be entered manually to ensure correct image aspect and scale bar sizing
- Spot-wise import requires at least one element to be present across the entirety of the sample with similar peak positions for each element
- Support for vendors outside Agilent, Perkin Elmer, and Thermo relies on flexible text image and file-per-line importers rather than native parameter reading

## Evidence

- [results] Supported vendor export formats enumerated: [evidence span withheld — anchor status: not_found]
- [readme] README: Agilent drag-and-drop import of batches with binary and csv support: [evidence span withheld — anchor status: not_found]
