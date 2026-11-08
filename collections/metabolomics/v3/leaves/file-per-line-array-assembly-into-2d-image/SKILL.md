---
name: file-per-line-array-assembly-into-2d-image
description: Use when raw LA-ICP-MS data arrive as generic one-file-per-csv line exports
  or vendor exports lacking a native importer, and the goal is direct drag-and-drop
  assembly into a 2D elemental image without preprocessing.
when_to_use_negative:
- Data already exist as a calibrated NumPy export or another fully assembled 2D image
  — re-running the importer would discard prior quantification work.
- Mass spectrometry imaging data in a format outside the supported set (e.g. no text
  image, file-per-line, imzML, or vendor binary equivalent) — Pew2's importers will
  not recognize it.
- The analysis requires interactive MSI processing typical of generic MSI tools rather
  than LA-ICP-MS line/spot data — Pew2 is LA-ICP-MS specific and its importers assume
  laser line/spot acquisition semantics.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: pewlib
  role: Python library that implements the import of file-per-line and vendor data
    and the assembly/export routines used for batch conversion
  repo: https://github.com/djdt/pewlib
- name: Pew2
  role: GUI providing drag-and-drop import of file-per-line, text image, and vendor
    exports into the internal image format
  repo: https://github.com/djdt/pewpew
- name: NumPy
  role: Backend library used for the assembled per-isotope 2D arrays and the internal
    NumPy archive format
- name: Python
  role: Language in which pewlib import/export and batch conversion scripts are written
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.1c02138
    title: 'Pew <sup>2</sup> : Open-Source Imaging Software for Laser Ablation–Inductively
      Coupled Plasma–Mass Spectrometry'
  source_content_hash: sha256:fad9cd2e67c078a15b7aff272e420f603de3e268703a97ac070a05a58b1b6df8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/djdt/pewlib
  tools:
  - NumPy
  - Perkin Elmer ICP-MS
  - Pew2
  - Python
  - pewlib
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# file-per-line-array-assembly-into-2d-image

## Summary

Converts file-per-line vendor exports (one CSV per acquired laser line) into the 2D per-isotope image arrays used by Pew2/pewlib, alongside its sibling text-image and vendor binary importers (Agilent .b batches, Perkin Elmer ELAN .xl, Thermo Qtegra/iCap CSV).

## When to use

Use when raw LA-ICP-MS data arrive as generic one-file-per-csv line exports or vendor exports lacking a native importer, and the goal is direct drag-and-drop assembly into a 2D elemental image without preprocessing.

## When NOT to use

- Data already exist as a calibrated NumPy export or another fully assembled 2D image — re-running the importer would discard prior quantification work.
- Mass spectrometry imaging data in a format outside the supported set (e.g. no text image, file-per-line, imzML, or vendor binary equivalent) — Pew2's importers will not recognize it.
- The analysis requires interactive MSI processing typical of generic MSI tools rather than LA-ICP-MS line/spot data — Pew2 is LA-ICP-MS specific and its importers assume laser line/spot acquisition semantics.

## Inputs

- File-per-line LA-ICP-MS export: one delimited CSV per acquired laser line
- 2D delimited text image (generic text image format for other vendors)
- Agilent MassHunter batch (.b directory, G7200B or later), binary or csv, one sample per line
- Perkin Elmer ELAN “XL” directory
- Thermo iCap CSV or LDR export / Thermo Qtegra export
- Iolite laser logs (ActiveView2 or Chromium3 style) for ablation-time-aligned sync
- imzML files (via import wizard)

## Outputs

- Pew2/pewlib internal image object: 2D per-isotope NumPy signal arrays with isotope names and instrument/laser configuration
- Elemental image displayed in the Pew2 GUI with correct image aspect and scale bar
- Batch-converted NumPy export (via pewlib script)

## How to apply

Detect whether the export is a one-file-per-csv line series, a 2D delimited text image, or a vendor binary/CSV format (Agilent MassHunter .b batch directory, Perkin Elmer ELAN “XL” directory, Thermo iCap CSV/LDR or Qtegra). Parse each per-line file to extract the per-isotope signal arrays, reading isotope names and available instrument parameters such as MS acquisition time from the export. Assemble the per-isotope line arrays into 2D image arrays ordered by laser pixel and line index, applying the laser spot size and scan speed (e.g. 10 μm beam, 40 μm/s, 0.25 s acquisition) so image aspect and scale bar are correct; enter laser spot size and speed manually after import when the export omits them. Package the arrays, isotope names, and configuration into the internal NumPy-based image format via pewlib, then load into the Pew2 GUI or call pewlib in a Python script for batch conversion.

## Related tools

- **pewlib** (Python library that implements the import of file-per-line and vendor data and the assembly/export routines used for batch conversion) — https://github.com/djdt/pewlib
- **Pew2** (GUI providing drag-and-drop import of file-per-line, text image, and vendor exports into the internal image format) — https://github.com/djdt/pewpew
- **NumPy** (Backend library used for the assembled per-isotope 2D arrays and the internal NumPy archive format)
- **Python** (Language in which pewlib import/export and batch conversion scripts are written)

## Examples

```
pip install pewlib; python -c "import pewlib; img = pewlib.io.load('calibration')"  # or drag-and-drop a MassHunter .b batch / CSV line folder into the Pew2 GUI
```

## Evaluation signals

- Imported image preserves the per-line pixel ordering from the raw export — spot-wise single-marker peak positions map to consistent column order across lines.
- Instrument parameters read from the export (e.g. MS acquisition time) appear correctly in the image configuration; scale bar and image aspect match the entered laser spot size and scan speed (e.g. 10 μm, 40 μm/s).
- Isotope/channel count and names in the assembled arrays match those declared in the vendor export or per-line CSV headers.
- Spot-wise import shows ≥ one marker element with similar peak positions across the whole sample; each element's per-line array integrates without missing peaks.
- Round-trip check: batch-converted NumPy export via pewlib loads back into Pew2 with identical per-isotope array values and configuration.

## Limitations

- Drag-and-drop vendor support is limited to MassHunter G7200B or later batches, Perkin Elmer ELAN “XL” directories, and Thermo Qtegra/iCap exports; older or other-vendor files must first be reshaped into text image or file-per-line format.
- Spot-wise import requires at least one element present across the entirety of the sample with similar peak positions for each element; otherwise integration fails.
- Laser spot size and speed are often not carried in the export and must be entered manually after import, or image aspect and scale bar sizing will be wrong.
- File-per-line import assumes one sample/line per file in the generic CSV importer; irregular line counts or interleaved files are not handled automatically.

## Evidence

- [abstract] Support for other vendors is maintained with a flexible text image (2D delimited text files) and file-per-line importer.: [evidence span withheld — anchor status: not_found]
- [abstract] Pew2 is LA-ICP-MS specific software; its scope statement in the abstract restricts use to LA-ICP-MS platforms.: [evidence span withheld — anchor status: not_found]
