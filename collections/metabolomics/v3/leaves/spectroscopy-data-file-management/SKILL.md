---
name: spectroscopy-data-file-management
description: Use when ingesting or storing spectroscopic data (1H/13C NMR, IR, MS)
  from JCAMP-DX or Bruker files and a FAIR-compliant three-file artifact set per spectrum
  is required.
when_to_use_negative:
- The spectrum is already stored as a processed feature table or peak list, so no
  original/edited/image artifact set is needed
- Only 2D NMR viewing or multi-spectrum comparison is required, which ChemSpectra
  does not yet support (listed as future work)
- The workflow only needs peak picking or signal integration output transferred to
  the ELN, not file artifact storage
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0091
tools:
- name: ChemSpectra
  role: Backend web service that converts, edits and re-composes spectra, producing
    the edited JCAMP-DX version and the png image
  repo: https://github.com/ComPlat/chem-spectra-app
- name: Chemotion ELN
  role: Host application whose file-management contract (original + last edited version
    + generated png per spectrum) the storage layout mirrors
  repo: https://github.com/ComPlat/chemotion_ELN
- name: react-spectra-editor
  role: Front-end editor whose jcampconverter library parses the composed JCAMP-DX
    file back into spectroscopic data for validation
  repo: https://github.com/ComPlat/react-spectra-editor
- name: chem-spectra-client
  role: Standalone page that calls the chem-spectra-app service directly for upload,
    edit and download of spectra
  repo: https://github.com/ComPlat/chem-spectra-client
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:0dd269119e5ee29196e4e5d8ee1851a4aec6323378ac22562141fc750014a6c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ComPlat/chem-spectra-app
  tools:
  - ChemSpectra
  - Chemotion ELN
  - JavaScript
  - chem-spectra-client
  - react-spectra-editor
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# spectroscopy-data-file-management

## Summary

Implements the ChemSpectra/Chemotion ELN file-management contract for spectroscopic data: for each uploaded spectrum (JCAMP-DX, Bruker, mzML, RAW), preserve the untouched original, store the last edited version, and save an automatically generated png image of the spectra.

## When to use

Use when ingesting or storing spectroscopic data (1H/13C NMR, IR, MS) from JCAMP-DX or Bruker files and a FAIR-compliant three-file artifact set per spectrum is required.

## When NOT to use

- The spectrum is already stored as a processed feature table or peak list, so no original/edited/image artifact set is needed
- Only 2D NMR viewing or multi-spectrum comparison is required, which ChemSpectra does not yet support (listed as future work)
- The workflow only needs peak picking or signal integration output transferred to the ELN, not file artifact storage

## Inputs

- Uploaded 1D NMR, IR or MS spectrum file in JCAMP-DX format (.jdx/.dx/.jcamp)
- Bruker NMR archive (zip) or proprietary FID/RAW files
- mzML MS file (converted to JCAMP-DX with pymzML if needed)

## Outputs

- Original spectrum file stored byte-identical to the upload (e.g. *.zip)
- Last-edited JCAMP-DX version (e.g. *.edit.jdx)
- Automatically generated png image of the spectra (e.g. *.edit.png)

## How to apply

Read the uploaded spectrum file (JCAMP-DX .jdx/.dx/.jcamp or a Bruker zip) as raw bytes so the unmodified original artifact is retained unchanged, mirroring chem-spectra-app's FileContainer handling. Send the same spectrum through the server-side conversion pipeline (the Flask backend of chem-spectra-app, built on nmrglue and SciPy) to compose an edited JCAMP-DX version, and through the image endpoint so the matplotlib-based composer renders an automatically generated png of the spectra. Write all three artifacts to disk: the untouched original bytes, the last-edited JCAMP-DX version, and the generated png, exactly as the Chemotion ELN integration stores them per uploaded spectrum. Rationale: the paper's file management supports 'storage of the original spectra along with the last edited version and an automatically generated image of the spectra in png format', keeping open formats (JCAMP-DX, mzML) for interoperability per FAIR principles.

## Related tools

- **ChemSpectra** (Backend web service that converts, edits and re-composes spectra, producing the edited JCAMP-DX version and the png image) — https://github.com/ComPlat/chem-spectra-app
- **Chemotion ELN** (Host application whose file-management contract (original + last edited version + generated png per spectrum) the storage layout mirrors) — https://github.com/ComPlat/chemotion_ELN
- **react-spectra-editor** (Front-end editor whose jcampconverter library parses the composed JCAMP-DX file back into spectroscopic data for validation) — https://github.com/ComPlat/react-spectra-editor
- **chem-spectra-client** (Standalone page that calls the chem-spectra-app service directly for upload, edit and download of spectra) — https://github.com/ComPlat/chem-spectra-client

## Examples

```
curl -F 'file=spectrum.jdx' -X POST https://chemspectra.example/jcamp && curl -F 'file=spectrum.jdx' -X POST https://chemspectra.example/image
```

## Evaluation signals

- The output directory contains exactly three files per uploaded spectrum: the original, an edited JCAMP-DX version, and a png image
- The stored original is byte-identical to the uploaded file (checksum comparison)
- The edited *.edit.jdx can be parsed back into spectroscopic data by the jcampconverter library used in react-spectra-editor
- The *.edit.png is a non-empty, renderable image of the spectra produced by the server-side composer
- Non-JCAMP-DX inputs (RAW, mzML, Bruker) have been converted to JCAMP-DX before storage, allowing standardized treatment of all data files

## Limitations

- ChemSpectra handles one-dimensional 1H and 13C NMR, IR and MS; viewing and analysis of 2D NMR data is not supported
- Processing of FID files needs improved phase and baseline correction, and further MS file formats are not yet supported
- Proprietary RAW (MS) files require conversion to mzML via msConvert (ProteoWizard) in a docker container before this pipeline applies
- The paper reports no quantitative evaluation or performance metrics for the file-management workflow

## Evidence

- [abstract] The file management supports the storage of the original spectra along with the last edited version and an automatically generated image of the spectra in png format: "The file management supports the storage of the original spectra along with the last edited version and an automatically generated image of the spectra in png format"
- [methods] The jcampconverter library inside the react-spectra-editor extracts spectroscopic data of the composed JCAMP-DX file: "The jcampconverter library [17] inside the react-spectra-editor extracts spectroscopic data of the composed JCAMP-DX file which is sent from the server-side"
- [discussion] All data files that are provided as non-JCAMP-DX files are processed and converted to JCAMP-DX: "All data files that are provided as non-JCAMP-DX files are processed and converted to JCAMP-DX, allowing a standardized treatment of all data files after a first processing step."
- [readme] This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files.: "This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files."
