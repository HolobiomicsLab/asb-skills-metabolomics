---
name: spectrum-type-mapping-table-csv-export
description: Use when a workflow ingests mixed spectroscopy files (JCAMP-DX, mzML,
  RAW, FID, or ZIP) and each file must be routed to the correct spectra type (NMR,
  IR, or MS) before downstream processing or type-specific analysis.
when_to_use_negative:
- Input is already a typed or labelled spectrum (e.g. the Chemotion ELN has already
  assigned a spectra type) — re-classification adds no information.
- The task is 2D NMR viewing/analysis or comparison of different spectra in one window
  — the article lists these as missing functions in ChemSpectra.
- Input spectra use formats ChemSpectra does not yet support (the article notes further
  MS file formats should be added; unsupported formats will fall out as 'unknown').
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3360
tools:
- name: ChemSpectra / chem-spectra-app
  role: Backend service that reads spectra and maps file extensions (jcamp, jdx, dx;
    mzML; RAW; FID/ZIP) to spectra types via the data_type.json mapping
  repo: https://github.com/ComPlat/chem-spectra-app
- name: NMRglue
  role: 'Python library used by JcampBaseConverter inside chem-spectra-app to read
    JCAMP-DX blocks for ##DATA TYPE detection'
- name: pymzML
  role: Python parser used to convert mzML to JCAMP-DX during the MS conversion route
    so the file is treated as a mass spectrum
- name: msconvert (ProteoWizard)
  role: Runs in a docker container called by chem-spectra-app to convert proprietary
    RAW MS files to mzML before classification
- name: react-spectra-editor
  role: Front-end editor that automatically extracts the spectra type from the provided
    files before display and editing
  repo: https://github.com/ComPlat/react-spectra-editor
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:790667f54a65d25a557cbcfe5e3b0e9d590e85a42720382ba6091506770a6052
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ComPlat/chem-spectra-app
  tools:
  - ChemSpectra
  - ChemSpectra / chem-spectra-app
  - Docker
  - NMRglue
  - Python
  - SciPy
  - msconvert (ProteoWizard)
  - pymzML
  - react-spectra-editor
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6842105263157895
n_claims: 19
---

# Spectrum type mapping table CSV export

## Summary

Classifies each uploaded JCAMP-DX, mzML, RAW, or FID spectrum as NMR, IR, or MS using chem-spectra-app's extension- and ##DATA TYPE-driven mapping, then records filename-to-type assignments in a spectrum_types.csv table.

## When to use

Use when a workflow ingests mixed spectroscopy files (JCAMP-DX, mzML, RAW, FID, or ZIP) and each file must be routed to the correct spectra type (NMR, IR, or MS) before downstream processing or type-specific analysis.

## When NOT to use

- Input is already a typed or labelled spectrum (e.g. the Chemotion ELN has already assigned a spectra type) — re-classification adds no information.
- The task is 2D NMR viewing/analysis or comparison of different spectra in one window — the article lists these as missing functions in ChemSpectra.
- Input spectra use formats ChemSpectra does not yet support (the article notes further MS file formats should be added; unsupported formats will fall out as 'unknown').

## Inputs

- JCAMP-DX spectra files (.jcamp, .jdx, .dx) for NMR, IR, or MS
- mzML mass spectrometry files
- proprietary RAW (MS) files
- FID (NMR) files
- ZIP archives of NMR spectra

## Outputs

- spectrum_types.csv with one row per input file and a type column (NMR, IR, MS, or unknown)

## How to apply

Collect the input spectra into a working directory and, for each JCAMP-DX file (extensions jcamp, jdx, dx), invoke the chem-spectra-app conversion chain in which JcampBaseConverter reads the file via nmrglue's read_blocks and the spectrum is classified from the first recognised ##DATA TYPE= block according to the data_type.json mapping. For mzML files run the MS conversion route (MSConverter/MSComposer, which converts mzML to JCAMP-DX using pymzML) so the file is treated as a mass spectrum, and for proprietary RAW inputs call the msconvert (ProteoWizard) docker sidecar to produce mzML before classification; FID and ZIP files map to NMR spectra. Record each filename together with its assigned type (NMR, IR, MS, or unknown) in a mapping table and write it to spectrum_types.csv. Validation: the table exists with one row per input file and every type value is one of NMR, IR, MS, or unknown.

## Related tools

- **ChemSpectra / chem-spectra-app** (Backend service that reads spectra and maps file extensions (jcamp, jdx, dx; mzML; RAW; FID/ZIP) to spectra types via the data_type.json mapping) — https://github.com/ComPlat/chem-spectra-app
- **NMRglue** (Python library used by JcampBaseConverter inside chem-spectra-app to read JCAMP-DX blocks for ##DATA TYPE detection)
- **pymzML** (Python parser used to convert mzML to JCAMP-DX during the MS conversion route so the file is treated as a mass spectrum)
- **msconvert (ProteoWizard)** (Runs in a docker container called by chem-spectra-app to convert proprietary RAW MS files to mzML before classification)
- **react-spectra-editor** (Front-end editor that automatically extracts the spectra type from the provided files before display and editing) — https://github.com/ComPlat/react-spectra-editor

## Examples

```
python -m chemspectra.file_convert.spectra_data_read --input spectra/ --map data_type.json --out spectrum_types.csv
```

## Evaluation signals

- spectrum_types.csv exists after the run and contains exactly one row per input file.
- Every row's type value is one of the allowed tokens: NMR, IR, MS, or unknown — no other labels appear.
- Files with extensions jcamp/jdx/dx resolve via the data_type.json mapping to NMR, IR, or MS from the first recognised ##DATA TYPE= block; mzML files always resolve to MS.
- Proprietary RAW files produce an mzML intermediate (via msconvert docker call) before classification rather than remaining untyped.
- FID and ZIP inputs are classified as NMR spectra, not MS.

## Limitations

- Files whose extension or ##DATA TYPE is not covered by data_type.json fall through as 'unknown'.
- The article notes chem-spectra-app should be extended to support further MS file formats, so some MS formats are not yet classified.
- FID file processing needs improved phase and baseline correction methods, so FID-derived NMR classification is a preprocessing-bound route.

## Evidence

- [full_text] react-spectra-editor maps file extensions to spectra types: [evidence span withheld — anchor status: not_found]
- [methods] chem-spectra-app reads JCAMP-DX via nmrglue and JcampBaseConverter: "The jcampconverter library [17] inside the react-spectra-editor extracts spectroscopic data of the composed JCAMP-DX file which is sent from the server-side"
- [results] mzML is converted to JCAMP-DX with pymzML: "mzML files are converted to JCAMP-DX using pymzML, an Open Source python mass spectrometry file parser."
- [results] msConvert in docker converts RAW to mzML: "MsConvert in a docker container is called by the chem-spectra-app to achieve this job."
- [abstract] Open formats JCAMP-DX and mzML are supported: "The software can deal with the open file formats JCAMP-DX (IR, MS, NMR) and mzML (MS) proposing these data file types to gain interoperable data"
