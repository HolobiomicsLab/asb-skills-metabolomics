---
name: spectra-server-service-operation
description: Use when an input spectrum is not already in JCAMP-DX (mzML MS, proprietary
  RAW, FID, Bruker, BagIt) and it must be standardized server-side to JCAMP-DX before
  visualization, editing, or transfer of signals to an ELN.
when_to_use_negative:
- The input file is already in JCAMP-DX format — it can be handled directly by the
  jcampconverter-based editor without the non-JCAMP-DX conversion step
- The goal is processing 2D NMR data — the editor does not yet support viewing and
  analysis of 2D data, which is stated as future work
- The spectrum is in an MS format not yet supported — the article notes chem-spectra-app
  should be extended to support further MS file formats, so unsupported formats fall
  outside this skill
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: ChemSpectra (chem-spectra-app)
  role: Flask-based backend service that receives non-JCAMP-DX files, standardizes
    them to JCAMP-DX as the first processing step, and serves subsequent spectra operations
  repo: https://github.com/ComPlat/chem-spectra-app
- name: pymzML
  role: Open source Python mass spectrometry file parser used to convert mzML files
    to JCAMP-DX
- name: MsConvert (ProteoWizard)
  role: Called in a docker container by chem-spectra-app to decode proprietary RAW
    MS files into mzML before JCAMP-DX conversion
- name: react-spectra-editor
  role: Client-side editor whose jcampconverter library extracts spectroscopic data
    from the composed JCAMP-DX file sent by the server
  repo: https://github.com/ComPlat/react-spectra-editor
- name: chem-spectra-client
  role: Standalone client page that calls the chem-spectra-app service directly to
    submit and retrieve spectra
  repo: https://github.com/ComPlat/chem-spectra-client
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:5a1f8c338e88f8f8317b247add2ef2dfd229a58369ac9ef6e6b426f57321650a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ComPlat/chem-spectra-app
  tools:
  - ChemSpectra
  - ChemSpectra (chem-spectra-app)
  - MsConvert (ProteoWizard)
  - chem-spectra-client
  - pymzML
  - react-spectra-editor
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# spectra-server-service-operation

## Summary

Server-side spectra handling in the chem-spectra-app backend: every uploaded non-JCAMP-DX analytical data file (e.g. mzML mass spectrometry, RAW, FID, Bruker) is decoded and converted to JCAMP-DX as a first processing step, so that all downstream reading, editing, peak-picking and download operations operate on one standardized format.

## When to use

Use when an input spectrum is not already in JCAMP-DX (mzML MS, proprietary RAW, FID, Bruker, BagIt) and it must be standardized server-side to JCAMP-DX before visualization, editing, or transfer of signals to an ELN.

## When NOT to use

- The input file is already in JCAMP-DX format — it can be handled directly by the jcampconverter-based editor without the non-JCAMP-DX conversion step
- The goal is processing 2D NMR data — the editor does not yet support viewing and analysis of 2D data, which is stated as future work
- The spectrum is in an MS format not yet supported — the article notes chem-spectra-app should be extended to support further MS file formats, so unsupported formats fall outside this skill

## Inputs

- mzML mass spectrometry file
- Proprietary MS RAW file (converted to mzML via MsConvert/ProteoWizard in a docker container)
- NMR FID file
- Bruker, BagIt and other non-JCAMP-DX spectra files handled by the backend

## Outputs

- JCAMP-DX spectrum file (IR, MS, or NMR) containing the spectral data extracted from the source file
- Standardized JCAMP-DX input for downstream ChemSpectra operations (editing, peak detection, PNG generation, ELN signal transfer)

## How to apply

Submit the file to the chem-spectra-app backend (a Flask-based Python service built on a modified NMRglue and SciPy) rather than trying to edit it directly. For proprietary MS RAW files, the backend first converts RAW to mzML by calling MsConvert (ProteoWizard) inside a docker container; mzML files are then converted to JCAMP-DX using pymzML, an open source Python mass spectrometry file parser. NMR FID processing follows the analogous non-JCAMP-DX-to-JCAMP-DX standardization path. Once the JCAMP-DX file is produced, all subsequent handling (upload, read, edit, download, peak detection, PNG image generation, signal transfer to Chemotion ELN) treats it like any other JCAMP-DX file. Verify success by checking that the retrieved output is a readable JCAMP-DX file containing the spectral data extracted from the original source.

## Related tools

- **ChemSpectra (chem-spectra-app)** (Flask-based backend service that receives non-JCAMP-DX files, standardizes them to JCAMP-DX as the first processing step, and serves subsequent spectra operations) — https://github.com/ComPlat/chem-spectra-app
- **pymzML** (Open source Python mass spectrometry file parser used to convert mzML files to JCAMP-DX)
- **MsConvert (ProteoWizard)** (Called in a docker container by chem-spectra-app to decode proprietary RAW MS files into mzML before JCAMP-DX conversion)
- **react-spectra-editor** (Client-side editor whose jcampconverter library extracts spectroscopic data from the composed JCAMP-DX file sent by the server) — https://github.com/ComPlat/react-spectra-editor
- **chem-spectra-client** (Standalone client page that calls the chem-spectra-app service directly to submit and retrieve spectra) — https://github.com/ComPlat/chem-spectra-client

## Evaluation signals

- The retrieved output is a readable JCAMP-DX mass-spectrometry file containing the spectral data extracted from the mzML (or RAW-derived mzML) source
- After conversion, the file is treated identically to native JCAMP-DX files by all backend operations (read, edit, download, peak detection, PNG image generation)
- For RAW inputs, an intermediate mzML file was successfully produced by the MsConvert docker container step before JCAMP-DX conversion
- The original spectra, the last edited version, and an automatically generated PNG image of the spectra are all stored by the file management layer

## Limitations

- Processing of FID files has to be improved by adding advanced phase correction and baseline correction methods
- The chem-spectra-app should be extended to support further MS file formats; only supported formats (JCAMP-DX, mzML, and the RAW/FID extensions) are handled
- ChemSpectra supports only one-dimensional 1H and 13C NMR; viewing and analysis of 2D data and comparison of different spectra in one window are listed as future improvements

## Evidence

- [results] Convert mzML files to JCAMP-DX using pymzML: "mzML files are converted to JCAMP-DX using pymzML, an Open Source python mass spectrometry file parser."
- [discussion] All non-JCAMP-DX files are standardized first: "All data files that are provided as non-JCAMP-DX files are processed and converted to JCAMP-DX, allowing a standardized treatment of all data files after a first processing step."
- [results] RAW MS files decoded via MsConvert docker container: "MsConvert in a docker container is called by the chem-spectra-app to achieve this job."
- [readme] README: supported input formats: "This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files."
