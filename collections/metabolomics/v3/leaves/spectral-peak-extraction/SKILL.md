---
name: spectral-peak-extraction
description: Use when a JCAMP-DX or mzML spectrum (IR, MS, or 1D 1H/13C NMR) must
  be reduced to a peak list, and peaks should be picked automatically relative to
  a default or user-defined intensity threshold.
when_to_use_negative:
- The spectrum is 2D NMR data — viewing and analysis of 2D data is explicitly listed
  as a missing signal not yet supported by ChemSpectra
- The input is a proprietary RAW MS or FID NMR file that has not yet been converted
  (RAW is first converted to mzML with msConvert in a docker container, and all non-JCAMP-DX
  files are converted to JCAMP-DX in a first processing step)
- The task is quantitative analysis or performance evaluation rather than peak list
  generation — the article reports no quantitative evaluation or performance metrics
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0031
- http://edamontology.org/topic_0593
tools:
- name: ChemSpectra
  role: Web-based spectra editor providing automatic peak detection according to a
    default or manually defined threshold for IR, MS, and 1D NMR data
  repo: https://github.com/ComPlat/chem-spectra-app
- name: react-spectra-editor
  role: Front-end editor whose jcampconverter library extracts spectroscopic data
    from the composed JCAMP-DX file for display and peak picking
  repo: https://github.com/ComPlat/react-spectra-editor
- name: chem-spectra-client
  role: Standalone client page that calls the chem-spectra-app service directly and
    realizes file management and peaks output
  repo: https://github.com/ComPlat/chem-spectra-client
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:4b0edc9d6a75ef9bfd93b017669414921792502a308c786b26c7be8844f852ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ComPlat/chem-spectra-app
  tools:
  - ChemSpectra
  - JavaScript
  - chem-spectra-client
  - react-spectra-editor
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0031
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spectral-peak-extraction

## Summary

Implements ChemSpectra's automatic peak detection, which selects spectral data points whose intensity exceeds a default or manually defined threshold and writes the detected peak positions and intensities as a structured peak list for JCAMP-DX (IR, MS, NMR) and mzML (MS) spectra.

## When to use

Use when a JCAMP-DX or mzML spectrum (IR, MS, or 1D 1H/13C NMR) must be reduced to a peak list, and peaks should be picked automatically relative to a default or user-defined intensity threshold.

## When NOT to use

- The spectrum is 2D NMR data — viewing and analysis of 2D data is explicitly listed as a missing signal not yet supported by ChemSpectra
- The input is a proprietary RAW MS or FID NMR file that has not yet been converted (RAW is first converted to mzML with msConvert in a docker container, and all non-JCAMP-DX files are converted to JCAMP-DX in a first processing step)
- The task is quantitative analysis or performance evaluation rather than peak list generation — the article reports no quantitative evaluation or performance metrics

## Inputs

- JCAMP-DX spectrum file (IR, MS, or 1D 1H/13C NMR)
- mzML file (MS), converted to JCAMP-DX using pymzML
- threshold parameter (default per spectra type or manually defined)

## Outputs

- peaks.json / structured peaks output file containing detected peak positions (x) and intensities (y)
- Peak list transferred to the ELN via the automated signal-transfer procedure

## How to apply

Load the spectrum file (JCAMP-DX for IR/MS/NMR, or mzML converted to JCAMP-DX via pymzML) and parse its data table into x/y data points. Read the threshold parameter, falling back to the default peak-detection threshold defined for the spectrum technique (NMR, IR, or MS) when none is supplied. Apply the automatic peak-detection logic over the parsed data points, selecting points whose intensity exceeds the threshold as detected peaks, as done in the chem-spectra-app backend which handles decoding and peak-picking of spectra files. Write the resulting list of detected peak positions (x) and intensities (y) to a structured peaks output file, from which detected or manually chosen signals can be transferred automatically to the ELN. The decision point is the threshold itself: raising it reduces noise-driven peaks, lowering it captures weaker signals.

## Related tools

- **ChemSpectra** (Web-based spectra editor providing automatic peak detection according to a default or manually defined threshold for IR, MS, and 1D NMR data) — https://github.com/ComPlat/chem-spectra-app
- **react-spectra-editor** (Front-end editor whose jcampconverter library extracts spectroscopic data from the composed JCAMP-DX file for display and peak picking) — https://github.com/ComPlat/react-spectra-editor
- **chem-spectra-client** (Standalone client page that calls the chem-spectra-app service directly and realizes file management and peaks output) — https://github.com/ComPlat/chem-spectra-client

## Evaluation signals

- The peaks.json file exists and is valid JSON
- Each detected peak is a list of objects with a numeric x (and y) entry, consistent with the peak structure ChemSpectra uses
- Every peak position corresponds to a data point whose intensity exceeds the applied threshold (default or manual)
- The peak list can be transferred to the ELN via the automated procedure without schema errors
- Running detection with the same threshold on the same input yields a reproducible, unchanged peak list

## Limitations

- Peak detection is purely threshold-based; choosing a threshold appropriate to the spectrum type (NMR, IR, MS) is the user's responsibility and drives which peaks are selected
- ChemSpectra currently lacks functions for viewing and analysis of 2D NMR data, comparison of different spectra in one application window, and advanced phase/baseline correction for FID processing
- Further MS file formats beyond JCAMP-DX and mzML are planned as future work, limiting which inputs can be peak-picked directly

## Evidence

- [discussion] automatic peak detection according to a default or manually defined threshold: "the automatic peak detection according to a default or manually defined threshold"
- [abstract] automated transfer of detected or manually chosen signals to the ELN: "an automated procedure for the transfer of the detected or manually chosen signals to the ELN was implemented"
- [methods] jcampconverter extracts spectroscopic data of the JCAMP-DX file: "The jcampconverter library [17] inside the react-spectra-editor extracts spectroscopic data of the composed JCAMP-DX file which is sent from the server-side"
- [readme] chem-spectra-client realizes peaks output: [evidence span withheld — anchor status: external_doc]
