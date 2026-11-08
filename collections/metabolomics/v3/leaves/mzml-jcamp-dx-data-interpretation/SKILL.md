---
name: mzml-jcamp-dx-data-interpretation
description: Use when the input is an mzML (MS) file that must be brought into JCAMP-DX
  form so it can undergo the ChemSpectra standardized processing, editing, and peak-output
  workflow.
when_to_use_negative:
- Input is already a JCAMP-DX (IR, MS, NMR) file — no non-JCAMP-DX conversion step
  is needed before processing
- Input is a proprietary RAW (MS) file and the msConvert (ProteoWizard) docker conversion
  to mzML has not yet been performed
- The goal is viewing or analysis of 2D NMR data — the article lists 2D data functions
  as missing, planned future work
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: ChemSpectra
  role: Flask-based backend (chem-spectra-app, built on Python with a modified NMRglue
    and SciPy) that decodes the mzML input and standardizes it to JCAMP-DX as the
    first processing step
  repo: https://github.com/ComPlat/chem-spectra-app
- name: pymzML
  role: Open Source python mass spectrometry file parser used inside chem-spectra-app
    to convert mzML files to JCAMP-DX
- name: msConvert (ProteoWizard)
  role: Upstream decoder called in a docker container by chem-spectra-app to convert
    proprietary RAW MS files to mzML before the JCAMP-DX conversion
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:f8faffc74a24415a080bdc6c88aea03654cbf8c33818e84046f7abd62f250502
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ComPlat/chem-spectra-app
  tools:
  - ChemSpectra
  - msConvert (ProteoWizard)
  - pymzML
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# mzml-jcamp-dx-data-interpretation

## Summary

Interprets mass spectrometry data held in mzML by leveraging the ChemSpectra chem-spectra-app backend, which converts non-JCAMP-DX files (mzML) to standardized JCAMP-DX using pymzML before any downstream spectral processing.

## When to use

Use when the input is an mzML (MS) file that must be brought into JCAMP-DX form so it can undergo the ChemSpectra standardized processing, editing, and peak-output workflow.

## When NOT to use

- Input is already a JCAMP-DX (IR, MS, NMR) file — no non-JCAMP-DX conversion step is needed before processing
- Input is a proprietary RAW (MS) file and the msConvert (ProteoWizard) docker conversion to mzML has not yet been performed
- The goal is viewing or analysis of 2D NMR data — the article lists 2D data functions as missing, planned future work

## Inputs

- mzML mass spectrometry file (non-JCAMP-DX open format)
- proprietary RAW (MS) file (only after prior msConvert RAW-to-mzML conversion)

## Outputs

- JCAMP-DX (MS) file containing the spectral data extracted from the mzML source
- readable JCAMP-DX mass-spectrometry file usable for standardized downstream ChemSpectra processing

## How to apply

Submit the mzML file to the chem-spectra-app backend, which decodes the input and converts it to JCAMP-DX using pymzML, an Open Source python mass spectrometry file parser; this conversion is deliberately placed as the first processing step so all subsequent data files receive standardized treatment. If the source is a proprietary RAW (MS) file instead of mzML, first decode it by converting RAW to mzML with msConvert (ProteoWizard) in a docker container called by chem-spectra-app, then proceed with the pymzML mzML-to-JCAMP-DX step. After conversion, the JCAMP-DX output can be loaded by the react-spectra-editor client, whose jcampconverter library extracts the spectroscopic data sent from the server-side for visualization, peak picking, and transfer of detected or manually chosen signals to the ELN or repository.

## Related tools

- **ChemSpectra** (Flask-based backend (chem-spectra-app, built on Python with a modified NMRglue and SciPy) that decodes the mzML input and standardizes it to JCAMP-DX as the first processing step) — https://github.com/ComPlat/chem-spectra-app
- **pymzML** (Open Source python mass spectrometry file parser used inside chem-spectra-app to convert mzML files to JCAMP-DX)
- **msConvert (ProteoWizard)** (Upstream decoder called in a docker container by chem-spectra-app to convert proprietary RAW MS files to mzML before the JCAMP-DX conversion)

## Evaluation signals

- Retrieve the generated output and verify it is a readable JCAMP-DX mass-spectrometry file
- Check the JCAMP-DX output contains the spectral data extracted from the mzML source
- Confirm the converted file undergoes standardized treatment together with other data files, since all non-JCAMP-DX inputs are converted to JCAMP-DX after a first processing step
- Verify the client-side jcampconverter library inside react-spectra-editor can extract the spectroscopic data of the composed JCAMP-DX file sent from the server-side

## Limitations

- The chem-spectra-app should be extended to support further MS file formats — additional MS format support is stated as future work
- Proprietary RAW (MS) files require an extra msConvert docker-container step before the mzML-to-JCAMP-DX conversion
- No quantitative evaluation, performance metrics, or user study results are reported for ChemSpectra
- Version numbers, release dates, and repository URLs for the ChemSpectra modules are not provided in the article

## Evidence

- [results] mzML converted to JCAMP-DX via pymzML: "mzML files are converted to JCAMP-DX using pymzML, an Open Source python mass spectrometry file parser."
- [discussion] non-JCAMP-DX files standardized first: "All data files that are provided as non-JCAMP-DX files are processed and converted to JCAMP-DX, allowing a standardized treatment of all data files after a first processing step."
- [results] RAW decoding via msConvert docker container: "MsConvert in a docker container is called by the chem-spectra-app to achieve this job."
- [readme] backend supported input formats: "This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files."
- [abstract] open formats for FAIR interoperability: "The software can deal with the open file formats JCAMP-DX (IR, MS, NMR) and mzML (MS) proposing these data file types to gain interoperable data"
