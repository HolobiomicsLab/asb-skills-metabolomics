---
name: nmr-peak-picking
description: Use when a 1D 1H/13C NMR spectrum (JCAMP-DX, or Bruker/FID after conversion)
  must be reduced to a machine-readable list of detected signal positions and intensities
  under a default or user-defined threshold.
when_to_use_negative:
- The spectrum is 2D NMR data — viewing and analysis of 2D data is explicitly listed
  as a missing signal not yet supported by ChemSpectra
- The input is an unprocessed FID requiring advanced phase or baseline correction
  — the article notes FID processing must still be improved by adding these methods
- The goal is quantitative integration, coupling-constant calculation, or multiplicity
  assignment rather than producing a peak list — these are separate NMR-specific functions
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ChemSpectra
  role: Backend web service (chem-spectra-app) that decodes the spectra files and
    performs the threshold-based peak picking
  repo: https://github.com/ComPlat/chem-spectra-app
- name: react-spectra-editor
  role: JavaScript frontend in which the user views the 1D NMR spectrum and adjusts
    the default or manual peak-detection threshold; its jcampconverter library extracts
    the spectroscopic data
  repo: https://github.com/ComPlat/react-spectra-editor
- name: chem-spectra-client
  role: Standalone client page that calls the chem-spectra-app service and realizes
    file management and peaks output
  repo: https://github.com/ComPlat/chem-spectra-client
- name: Chemotion ELN
  role: Main client that receives the automatically detected or manually chosen signals
    via the transfer workflow
  repo: https://github.com/ComPlat/chemotion_ELN
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:7a2ba4abc034f492b2fcf2794e3755bd8eea8ab233b68551f43c9162c2c38fcf
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
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# nmr-peak-picking

## Summary

Threshold-based automatic peak detection for 1D 1H/13C NMR spectra as implemented in ChemSpectra: peaks whose intensities exceed a default or manually defined threshold are extracted from parsed JCAMP-DX data points and written as a structured x/y peak list.

## When to use

Use when a 1D 1H/13C NMR spectrum (JCAMP-DX, or Bruker/FID after conversion) must be reduced to a machine-readable list of detected signal positions and intensities under a default or user-defined threshold.

## When NOT to use

- The spectrum is 2D NMR data — viewing and analysis of 2D data is explicitly listed as a missing signal not yet supported by ChemSpectra
- The input is an unprocessed FID requiring advanced phase or baseline correction — the article notes FID processing must still be improved by adding these methods
- The goal is quantitative integration, coupling-constant calculation, or multiplicity assignment rather than producing a peak list — these are separate NMR-specific functions

## Inputs

- 1D 1H or 13C NMR spectrum in JCAMP-DX format (data table of x/y points)
- NMR spectrum in Bruker/FID format (converted to JCAMP-DX by chem-spectra-app before peak picking)
- Peak-detection threshold value (optional; falls back to the NMR default threshold)

## Outputs

- Structured list of detected peaks with numeric x (position) and y (intensity) values, e.g. a peaks output file such as peaks.json
- Detected signals available for automated transfer to the Chemotion ELN / repository

## How to apply

Load the NMR spectrum file (JCAMP-DX preferred; Bruker FID/RAW can be converted first by the chem-spectra-app backend) and parse its data table into x/y data points using the jcampconverter/nmrglue-based reading approach of the chem-spectra-app converters. Read the threshold parameter: use the explicitly supplied value, or fall back to the per-spectra-type default peak-detection threshold defined for NMR when none is given. Apply the automatic peak-detection logic of the TechniqueComposer peak extraction, selecting data points whose intensity exceeds the threshold as detected peaks (the user can adjust this threshold per spectrum type in the react-spectra-editor UI). Write the resulting detected peak positions (x) and intensities (y) as a structured peaks output, which the automated workflow can transfer directly to the Chemotion ELN or repository.

## Related tools

- **ChemSpectra** (Backend web service (chem-spectra-app) that decodes the spectra files and performs the threshold-based peak picking) — https://github.com/ComPlat/chem-spectra-app
- **react-spectra-editor** (JavaScript frontend in which the user views the 1D NMR spectrum and adjusts the default or manual peak-detection threshold; its jcampconverter library extracts the spectroscopic data) — https://github.com/ComPlat/react-spectra-editor
- **chem-spectra-client** (Standalone client page that calls the chem-spectra-app service and realizes file management and peaks output) — https://github.com/ComPlat/chem-spectra-client
- **Chemotion ELN** (Main client that receives the automatically detected or manually chosen signals via the transfer workflow) — https://github.com/ComPlat/chemotion_ELN

## Evaluation signals

- The peaks output file (e.g. peaks.json) exists and is valid JSON
- Each detected peak is a list/object entry carrying a numeric x (position) and a numeric y (intensity) value
- Every emitted peak corresponds to a data point whose intensity exceeds the applied threshold (default NMR threshold or the manually set value)
- The peak list round-trips: detected signals are accepted by the automated transfer into the Chemotion ELN without format errors
- Peak positions visually coincide with signal maxima when the spectrum and peak list are rendered together in react-spectra-editor

## Limitations

- Only 1D 1H and 13C NMR are covered; viewing and analysis of 2D NMR data is planned future work
- Processing of FID files still needs advanced phase correction and baseline correction methods
- The automatic detection is purely threshold-based, so sensitivity depends entirely on the chosen default or manual threshold value

## Evidence

- [discussion] the automatic peak detection according to a default or manually defined threshold: "the automatic peak detection according to a default or manually defined threshold"
- [methods] The jcampconverter library extracts spectroscopic data of the composed JCAMP-DX file: "The jcampconverter library [17] inside the react-spectra-editor extracts spectroscopic data of the composed JCAMP-DX file which is sent from the server-side"
- [abstract] an automated procedure for the transfer of the detected or manually chosen signals to the ELN was implemented: "an automated procedure for the transfer of the detected or manually chosen signals to the ELN was implemented"
- [discussion] a workflow to write the automatically detected or manually chosen signals was implemented: "a workflow to write the automatically detected or manually chosen signals was implemented. This allows the direct transfer of information to e.g. the ELN or repository."
- [readme] README: backend web service processes spectra including NMR from JCAMP-DX and other formats: "This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files."
