---
name: threshold-based-peak-detection
description: Use when a JCAMP-DX or mzML spectrum (NMR, IR, MS) needs its peaks extracted
  automatically and the user has no better model than an intensity threshold; triggered
  whenever ChemSpectra's peak-picking dialog is invoked with a default or adjusted
  threshold.
when_to_use_negative:
- The peaks are to be chosen manually by the user rather than automatically — ChemSpectra
  also implements a workflow to write manually chosen signals, which should be used
  instead.
- The input is a non-JCAMP-DX format other than mzML or MS RAW (e.g. further unsupported
  MS formats) that the chem-spectra-app converter chain does not yet handle.
- 2D NMR data is involved — viewing and analysis of 2D data is explicitly listed as
  a missing signal/future improvement.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_1317
- http://edamontology.org/topic_3520
tools:
- name: ChemSpectra
  role: Web-based spectra editor whose backend (chem-spectra-app) performs the threshold-based
    peak-picking on the decoded spectra
  repo: https://github.com/ComPlat/chem-spectra-app
- name: chem-spectra-client
  role: Standalone client page that calls the chem-spectra-app service; realizes peaks
    output and file management
  repo: https://github.com/ComPlat/chem-spectra-client
- name: react-spectra-editor
  role: Front-end editor whose jcampconverter library extracts spectroscopic data
    of the composed JCAMP-DX file sent from the server-side
  repo: https://github.com/ComPlat/react-spectra-editor
- name: pymzML
  role: Open Source python mass spectrometry file parser used to convert mzML files
    to JCAMP-DX before peak detection
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:758c262502ea2a024d2f59dcbb9da0a24ed0e19d2432fbd4bdfe14704f95db71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ComPlat/chem-spectra-app
  tools:
  - ChemSpectra
  - JavaScript
  - chem-spectra-client
  - pymzML
  - react-spectra-editor
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_1317
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# threshold-based-peak-detection

## Summary

Automatic peak detection in ChemSpectra that selects spectroscopic signals by comparing data-point intensities against a per-technique default or manually defined threshold, then writes the detected peaks as a structured list for transfer to the ELN or repository.

## When to use

Use when a JCAMP-DX or mzML spectrum (NMR, IR, MS) needs its peaks extracted automatically and the user has no better model than an intensity threshold; triggered whenever ChemSpectra's peak-picking dialog is invoked with a default or adjusted threshold.

## When NOT to use

- The peaks are to be chosen manually by the user rather than automatically — ChemSpectra also implements a workflow to write manually chosen signals, which should be used instead.
- The input is a non-JCAMP-DX format other than mzML or MS RAW (e.g. further unsupported MS formats) that the chem-spectra-app converter chain does not yet handle.
- 2D NMR data is involved — viewing and analysis of 2D data is explicitly listed as a missing signal/future improvement.

## Inputs

- JCAMP-DX spectrum file (NMR, IR or MS)
- mzML MS file (converted to JCAMP-DX via pymzML)
- proprietary MS RAW file (converted via msConvert/ProteoWizard in a docker container)
- threshold parameter (per-spectra-type default if omitted)

## Outputs

- peaks.json — structured list of detected peaks, each a list of objects with numeric x (position) and y (intensity) entries
- detected signals written for direct transfer to the Chemotion ELN or repository

## How to apply

Load the spectrum and parse its data table into x/y data points using the chem-spectra-app converters (the nmrglue-based JCAMP block reader; mzML files are first converted to JCAMP-DX with pymzML, and proprietary MS RAW files via msConvert in a docker container). Read the threshold parameter, falling back to the default peak-detection threshold defined for the spectrum technique (NMR, IR or MS) when none is supplied. Apply the MSComposer/TechniqueComposer peak-extraction logic, selecting points whose intensity exceeds the threshold as detected peaks. Write the resulting peak positions (x) and intensities (y) as a structured peaks list, which the workflow can then transfer automatically to the Chemotion ELN or repository. The user may adjust the per-spectra-type default threshold before extraction if the automatic selection misses real signals or picks noise.

## Related tools

- **ChemSpectra** (Web-based spectra editor whose backend (chem-spectra-app) performs the threshold-based peak-picking on the decoded spectra) — https://github.com/ComPlat/chem-spectra-app
- **chem-spectra-client** (Standalone client page that calls the chem-spectra-app service; realizes peaks output and file management) — https://github.com/ComPlat/chem-spectra-client
- **react-spectra-editor** (Front-end editor whose jcampconverter library extracts spectroscopic data of the composed JCAMP-DX file sent from the server-side) — https://github.com/ComPlat/react-spectra-editor
- **pymzML** (Open Source python mass spectrometry file parser used to convert mzML files to JCAMP-DX before peak detection)

## Evaluation signals

- peaks.json exists and is valid JSON.
- Each detected peak in peaks.json is a list of objects with a numeric x (and y) entry, consistent with the peak structure ChemSpectra uses.
- Every reported peak x-position corresponds to a data point whose y intensity exceeds the applied threshold (default or user-supplied).
- Detected peaks match the signals written by the automated procedure that transfers detected or manually chosen signals to the ELN.
- With a sufficiently high threshold, the peak list shrinks to the strongest signals (e.g. the 1H/13C NMR reference signals), confirming threshold semantics.

## Limitations

- Only one-dimensional 1H and 13C NMR spectra are supported; viewing and analysis of 2D data is future work.
- No quantitative evaluation, performance metrics, or user study results are reported for the automatic peak detection.
- Support for further MS file formats is planned; non-JCAMP-DX files must first be converted to JCAMP-DX for standardized treatment.
- Processing of FID files needs improved phase correction and baseline correction methods, which affects peak quality for FID-derived spectra.

## Evidence

- [discussion] the automatic peak detection according to a default or manually defined threshold: "the automatic peak detection according to a default or manually defined threshold"
- [discussion] a workflow to write the automatically detected or manually chosen signals was implemented: "a workflow to write the automatically detected or manually chosen signals was implemented. This allows the direct transfer of information to e.g. the ELN or repository."
- [abstract] an automated procedure for the transfer of the detected or manually chosen signals to the ELN was implemented: "an automated procedure for the transfer of the detected or manually chosen signals to the ELN was implemented"
- [results] mzML files are converted to JCAMP-DX using pymzML: "mzML files are converted to JCAMP-DX using pymzML, an Open Source python mass spectrometry file parser."
- [readme] File management, peaks output, and notifications are realized in this server.: [evidence span withheld — anchor status: external_doc]
