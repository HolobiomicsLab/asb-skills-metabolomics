---
name: mass-spectrometry-file-format-conversion
description: Use when a mass spectrometry input file is not JCAMP-DX (e.g. mzML or
  vendor RAW) and downstream ChemSpectra/ELN processing requires the standardized
  JCAMP-DX representation of the spectral data.
when_to_use_negative:
- Input is already a JCAMP-DX (IR, MS, or NMR) file — the article states standardization
  is only needed for files 'provided as non-JCAMP-DX files'.
- Input is an NMR FID or other non-MS proprietary format that this MS conversion path
  (RAW→mzML→JCAMP-DX) does not cover.
- A different MS file format beyond RAW/mzML is required and not yet supported — the
  paper lists support for further MS file formats as future work.
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_2258
tools:
- name: ChemSpectra (chem-spectra-app)
  role: Flask-based backend that invokes the standardization step, calling MsConvert
    in a docker container for RAW files and converting mzML to JCAMP-DX via pymzML
  repo: github.com/ComPlat/chem-spectra-app
- name: pymzML
  role: Open Source python mass spectrometry file parser used to convert mzML files
    to JCAMP-DX
- name: MsConvert (ProteoWizard)
  role: Runs in a docker container called by chem-spectra-app to decode proprietary
    RAW MS files into mzML
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:35279b74ab830b388ab6c3a69f3d0ec2573655b603e48750e17d383e0f44d367
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemSpectra
  - ChemSpectra (chem-spectra-app)
  - MsConvert (ProteoWizard)
  - pymzML
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# mass-spectrometry-file-format-conversion

## Summary

Standardizes non-JCAMP-DX mass spectrometry inputs (mzML, and proprietary RAW via an mzML intermediate) into JCAMP-DX using pymzML inside the chem-spectra-app backend, so all downstream spectra handling operates on one standardized format.

## When to use

Use when a mass spectrometry input file is not JCAMP-DX (e.g. mzML or vendor RAW) and downstream ChemSpectra/ELN processing requires the standardized JCAMP-DX representation of the spectral data.

## When NOT to use

- Input is already a JCAMP-DX (IR, MS, or NMR) file — the article states standardization is only needed for files 'provided as non-JCAMP-DX files'.
- Input is an NMR FID or other non-MS proprietary format that this MS conversion path (RAW→mzML→JCAMP-DX) does not cover.
- A different MS file format beyond RAW/mzML is required and not yet supported — the paper lists support for further MS file formats as future work.

## Inputs

- mzML mass spectrometry file (non-JCAMP-DX input)
- proprietary RAW mass spectrometry file (converted to mzML first via MsConvert/ProteoWizard)

## Outputs

- JCAMP-DX mass spectrometry file containing the spectral data from the source file

## How to apply

First check the input class: JCAMP-DX files skip this step entirely, while all non-JCAMP-DX files are processed and converted to JCAMP-DX as a first processing step before any editing or peak analysis. For vendor RAW MS files, run an upstream conversion in which MsConvert (ProteoWizard) inside a docker container is called by the chem-spectra-app to produce mzML. Then convert the mzML file to JCAMP-DX using pymzML, an Open Source python mass spectrometry file parser, as implemented by the chem-spectra-app backend. Retrieve the generated JCAMP-DX output and confirm it is a readable JCAMP-DX mass-spectrometry file carrying the spectral data extracted from the mzML source; only after this standardization should spectra be loaded into the react-spectra-editor client for visualization, peak picking, or signal transfer to the ELN.

## Related tools

- **ChemSpectra (chem-spectra-app)** (Flask-based backend that invokes the standardization step, calling MsConvert in a docker container for RAW files and converting mzML to JCAMP-DX via pymzML) — github.com/ComPlat/chem-spectra-app
- **pymzML** (Open Source python mass spectrometry file parser used to convert mzML files to JCAMP-DX)
- **MsConvert (ProteoWizard)** (Runs in a docker container called by chem-spectra-app to decode proprietary RAW MS files into mzML)

## Evaluation signals

- The output parses as a readable JCAMP-DX mass-spectrometry file (accepted by the jcampconverter library in react-spectra-editor).
- The JCAMP-DX output contains the spectral data extracted from the mzML source (peak lists/intensities match the original spectrum).
- For RAW inputs, an intermediate mzML was successfully produced by the MsConvert docker step before JCAMP-DX conversion.
- All subsequent ChemSpectra processing (visualization, peak detection, ELN transfer) operates on the standardized JCAMP-DX file, confirming the first processing step succeeded.

## Limitations

- The article notes chem-spectra-app should be extended to support further MS file formats beyond those handled, so unsupported MS formats will fail this conversion path.
- Conversion of proprietary RAW depends on MsConvert running in a docker container; RAW and FID support is described as an extension ('ChemSpectra can be extended to read also other formats').
- No quantitative evaluation or performance metrics are reported for the conversion workflow in the article.

## Evidence

- [results] mzML to JCAMP-DX via pymzML: "mzML files are converted to JCAMP-DX using pymzML, an Open Source python mass spectrometry file parser."
- [discussion] non-JCAMP-DX standardization step: "All data files that are provided as non-JCAMP-DX files are processed and converted to JCAMP-DX, allowing a standardized treatment of all data files after a first processing step."
- [results] RAW to mzML upstream conversion: "MsConvert in a docker container is called by the chem-spectra-app to achieve this job."
- [readme] backend input formats: "This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files."
