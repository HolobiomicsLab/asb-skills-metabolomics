---
name: spectrum-technique-descriptor-lookup
description: Use when an uploaded spectrum file (jcamp, jdx, dx, mzML, RAW, FID, ZIP)
  must be classified into NMR, IR, MS, UV/VIS, XRD, CV or LC/MS before downstream
  editing, conversion or ELN ingestion.
when_to_use_negative:
- The input is already a feature table or peak list, not an original spectrum file
- 2D NMR data are provided — chem-spectra-app does not yet support viewing/analysis
  of 2D spectra
- The file format is one of the unsupported MS types (chem-spectra-app should be extended
  to support further MS file formats)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3314
tools:
- name: ChemSpectra
  role: backend web service whose data_type.json mapping assigns each uploaded file
    to a spectra type
  repo: github.com/ComPlat/chem-spectra-app
- name: NMRglue
  role: reads JCAMP-DX blocks consumed by JcampBaseConverter during classification
- name: pymzML
  role: converts mzML files to JCAMP-DX so they can be classified by the same backend
    chain
- name: msconvert (ProteoWizard)
  role: docker sidecar that converts proprietary RAW MS files to mzML before classification
- name: react-spectra-editor
  role: client-side editor that automatically extracts the spectra type from the provided
    files
  repo: github.com/ComPlat/react-spectra-editor
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:19547b0754df291c6629129f78b3aa6ec4995ea46e195180608d8450ef4095a5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemSpectra
  - Docker
  - NMRglue
  - Python
  - SciPy
  - msconvert (ProteoWizard)
  - pymzML
  - react-spectra-editor
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# spectrum-technique-descriptor-lookup

## Summary

A skill for determining which spectroscopic technique (NMR, IR, MS, UV/VIS, XRD, CV) a JCAMP-DX or mzML file encodes, using the chem-spectra-app backend's data-type mapping and the react-spectra-editor's automatic extraction.

## When to use

Use when an uploaded spectrum file (jcamp, jdx, dx, mzML, RAW, FID, ZIP) must be classified into NMR, IR, MS, UV/VIS, XRD, CV or LC/MS before downstream editing, conversion or ELN ingestion.

## When NOT to use

- The input is already a feature table or peak list, not an original spectrum file
- 2D NMR data are provided — chem-spectra-app does not yet support viewing/analysis of 2D spectra
- The file format is one of the unsupported MS types (chem-spectra-app should be extended to support further MS file formats)

## Inputs

- JCAMP-DX files (.jcamp, .jdx, .dx) for NMR, IR or MS
- mzML mass-spectrometry files
- proprietary RAW MS files
- FID or ZIP NMR files

## Outputs

- spectrum_types.csv mapping each input filename to its assigned spectra type (NMR, IR, MS, UV/VIS, XRD, CV, LC/MS or unknown)
- per-file classification labels used by react-spectra-editor and chem-spectra-app

## How to apply

Collect the incoming files and run each through the chem-spectra-app conversion chain: JCAMP-DX files are decoded with the JcampBaseConverter over nmrglue's read_blocks, and the spectrum type is read from the first recognised ##DATA TYPE= block via the data_type.json mapping (jcamp/jdx/dx map to NMR, IR or MS; mzML and RAW map to MS; FID or ZIP map to NMR). For proprietary RAW mass-spectrometry inputs, first call the msConvert (ProteoWizard) docker sidecar to produce mzML, then convert mzML to JCAMP-DX with pymzML so the file is handled by the same standardized route. Record each filename with its assigned technique label (NMR, IR, MS, UV/VIS, XRD, CV, LC/MS or unknown) in a mapping table such as spectrum_types.csv; the react-spectra-editor performs the equivalent automatic extraction on the client side for visualisation, so the backend table and client-side label must agree.

## Related tools

- **ChemSpectra** (backend web service whose data_type.json mapping assigns each uploaded file to a spectra type) — github.com/ComPlat/chem-spectra-app
- **NMRglue** (reads JCAMP-DX blocks consumed by JcampBaseConverter during classification)
- **pymzML** (converts mzML files to JCAMP-DX so they can be classified by the same backend chain)
- **msconvert (ProteoWizard)** (docker sidecar that converts proprietary RAW MS files to mzML before classification)
- **react-spectra-editor** (client-side editor that automatically extracts the spectra type from the provided files) — github.com/ComPlat/react-spectra-editor

## Examples

```
curl -F "file=@sample.jdx" http://localhost:3000/api/v1/chemspectra/files/search_jcamp --output spectrum_types.csv
```

## Evaluation signals

- spectrum_types.csv exists with one row per input file
- each type value is one of NMR, IR, MS, UV/VIS, XRD, CV, LC/MS, or unknown
- the backend label agrees with the spectra type automatically extracted by react-spectra-editor for the same file
- RAW files only classified after a successful msConvert-to-mzML conversion
- mzML files classified as MS after the pymzML-to-JCAMP-DX conversion step

## Limitations

- 2D NMR viewing/analysis is not yet supported
- FID processing needs improved phase and baseline correction methods
- further MS file formats still need to be supported by chem-spectra-app
- proprietary formats other than RAW/FID are handled only after extension of the converter chain

## Evidence

- [results] JCAMP-DX and mzML files map to spectra types: [evidence span withheld — anchor status: not_found]
- [results] data type mapping via data_type.json: [evidence span withheld — anchor status: not_found]
- [results] mzML conversion chain: "mzML files are converted to JCAMP-DX using pymzML, an Open Source python mass spectrometry file parser."
- [results] RAW conversion via docker: "MsConvert in a docker container is called by the chem-spectra-app to achieve this job."
- [intro] automatic extraction by editor: [evidence span withheld — anchor status: not_found]
- [discussion] standardized JCAMP-DX treatment: "All data files that are provided as non-JCAMP-DX files are processed and converted to JCAMP-DX, allowing a standardized treatment of all data files after a first processing step."
- [abstract] FAIR file formats: "The software can deal with the open file formats JCAMP-DX (IR, MS, NMR) and mzML (MS) proposing these data file types to gain interoperable data"
