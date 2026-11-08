---
name: ir-peak-detection-thresholding
description: Use when an IR spectrum in JCAMP-DX format needs its signals identified
  and reported as a peak table with bracketed intensity abbreviations, e.g. for transfer
  to an ELN.
when_to_use_negative:
- The spectrum is NMR or MS rather than IR — NMR requires reference-signal definition,
  integration, coupling constants and multiplicity assignment, not vw/w/m/s/vs intensity
  labels.
- The input is a proprietary RAW (MS) or FID (NMR) file that must first be converted
  (msConvert/pymzML to mzML/JCAMP-DX) before peak picking applies.
- 2D NMR data is involved — viewing/analysis of 2D data is explicitly listed as missing
  from ChemSpectra.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: ChemSpectra
  role: Web-based editor/service performing automatic IR peak detection with a default
    or manually defined threshold and writing annotated peaks for transfer to the
    ELN
  repo: https://github.com/ComPlat/chem-spectra-app
- name: react-spectra-editor
  role: Front-end that displays/edits the IR spectrum; its jcampconverter library
    extracts the JCAMP-DX spectroscopic data used for peak picking
  repo: https://github.com/ComPlat/react-spectra-editor
- name: chem-spectra-client
  role: Standalone client page calling the chem-spectra-app service; handles peaks
    output for IR spectra
  repo: https://github.com/ComPlat/chem-spectra-client
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:a233323fb697289a7a83e0b3a3f875260a3ed3dd9185316c1e5f3f00dedccf8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ComPlat/chem-spectra-app
  tools:
  - ChemSpectra
  - JavaScript
  - chem-spectra-client
  - react-spectra-editor
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# ir-peak-detection-thresholding

## Summary

Detects peaks in IR spectra using ChemSpectra's automatic threshold-based peak-picking (default or manually defined threshold) and annotates each detected peak with its wavenumber plus an IR intensity label (vw, w, m, s, vs) in square brackets.

## When to use

Use when an IR spectrum in JCAMP-DX format needs its signals identified and reported as a peak table with bracketed intensity abbreviations, e.g. for transfer to an ELN.

## When NOT to use

- The spectrum is NMR or MS rather than IR — NMR requires reference-signal definition, integration, coupling constants and multiplicity assignment, not vw/w/m/s/vs intensity labels.
- The input is a proprietary RAW (MS) or FID (NMR) file that must first be converted (msConvert/pymzML to mzML/JCAMP-DX) before peak picking applies.
- 2D NMR data is involved — viewing/analysis of 2D data is explicitly listed as missing from ChemSpectra.

## Inputs

- IR spectrum in JCAMP-DX format (x/y data points)
- peak-detection threshold (default or manually defined)

## Outputs

- List of detected IR peak positions (wavenumbers)
- Annotated peak table with wavenumber + bracketed intensity label (vw/w/m/s/vs), e.g. IR_peak_labels.txt

## How to apply

Load the IR JCAMP-DX file (ChemSpectra handles JCAMP-DX for IR; jcampconverter inside react-spectra-editor extracts the spectroscopic data server-side) and extract the x/y data points. Run automatic peak detection according to a default or manually defined threshold to obtain the list of detected IR peak positions; raise or lower the threshold if peaks are missed or noise is picked up. Classify each detected peak into one of the five intensity categories (very weak, weak, medium, strong, very strong) based on its relative intensity within the spectrum. Annotate each peak with its wavenumber followed by the intensity abbreviation in square brackets, e.g. [w], matching the format ChemSpectra writes into the JCAMP peak table, and write the annotated peaks out (e.g. IR_peak_labels.txt) for transfer to the ELN or repository.

## Related tools

- **ChemSpectra** (Web-based editor/service performing automatic IR peak detection with a default or manually defined threshold and writing annotated peaks for transfer to the ELN) — https://github.com/ComPlat/chem-spectra-app
- **react-spectra-editor** (Front-end that displays/edits the IR spectrum; its jcampconverter library extracts the JCAMP-DX spectroscopic data used for peak picking) — https://github.com/ComPlat/react-spectra-editor
- **chem-spectra-client** (Standalone client page calling the chem-spectra-app service; handles peaks output for IR spectra) — https://github.com/ComPlat/chem-spectra-client

## Evaluation signals

- Every detected IR peak in the output carries exactly one of the five bracketed intensity labels: [vw], [w], [m], [s], [vs].
- Each annotated wavenumber corresponds to a local maximum in the extracted x/y data that exceeds the chosen detection threshold.
- The annotated peak table is written to the expected output file (e.g. IR_peak_labels.txt) in the JCAMP peak-table format ChemSpectra uses.
- Annotated peaks can be transferred to the ELN via the automated signal-transfer workflow without loss of wavenumber or intensity information.

## Limitations

- The threshold is heuristic: a default or manually defined threshold controls detection, so very weak true signals below the threshold are missed and noise near the threshold may be annotated.
- ChemSpectra is positioned as a basic editor; 2D data viewing/analysis, spectra comparison in one window, and advanced phase/baseline correction (relevant to FID processing) are not supported, which can degrade peak detection on poorly corrected spectra.
- No quantitative evaluation or performance metrics for the peak detection are reported in the article.

## Evidence

- [discussion] Automatic peak detection according to a default or manually defined threshold: "the automatic peak detection according to a default or manually defined threshold"
- [results] The IR editor annotates detected IR peaks by adding the intensity of the identified signals (vw, w, m, s, vs) to the recorded wavenumber, placing the information in brackets after the corresponding wavenumber per IR reporting standards.: [evidence span withheld — anchor status: not_found]
- [methods] jcampconverter extracts JCAMP-DX data inside react-spectra-editor: "The jcampconverter library [17] inside the react-spectra-editor extracts spectroscopic data of the composed JCAMP-DX file which is sent from the server-side"
- [abstract] ChemSpectra handles open JCAMP-DX format for IR, MS, NMR: "The software can deal with the open file formats JCAMP-DX (IR, MS, NMR) and mzML (MS) proposing these data file types to gain interoperable data"
- [discussion] A workflow writes automatically detected or manually chosen signals for direct transfer to the ELN or repository.: "a workflow to write the automatically detected or manually chosen signals was implemented. This allows the direct transfer of information to e.g. the ELN or repository."
