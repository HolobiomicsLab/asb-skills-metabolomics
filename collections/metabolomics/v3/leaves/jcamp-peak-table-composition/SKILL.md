---
name: jcamp-peak-table-composition
description: Use when detected peak positions must be converted into an annotated
  peak table/label file (e.g. IR wavenumbers with bracketed vw/w/m/s/vs intensity
  labels) for export as JCAMP-DX metadata or ELN transfer.
when_to_use_negative:
- Input is a raw or proprietary file (e.g. MS RAW or NMR FID) that has not yet been
  converted to JCAMP-DX — convert first (RAW to mzML via msConvert, mzML to JCAMP-DX
  via pymzML) before composing the peak table
- The analysis goal is NMR-specific reporting such as integration, coupling constants
  or multiplicity assignment rather than peak-position/label annotation
- 2D NMR data — viewing and analysis of 2D data is not supported by ChemSpectra
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: ChemSpectra
  role: Backend service performing automatic peak detection (default or manual threshold)
    and writing detected or manually chosen signals into the exportable peak table
  repo: https://github.com/ComPlat/chem-spectra-app
- name: react-spectra-editor
  role: JavaScript front end that extracts spectroscopic data from JCAMP-DX via jcampconverter
    and hosts the editor/control panel used to pick or adjust peaks before table composition
  repo: https://github.com/ComPlat/react-spectra-editor
- name: chem-spectra-client
  role: Standalone client that integrates react-spectra-editor and realizes peaks
    output and file management
  repo: https://github.com/ComPlat/chem-spectra-client
- name: Chemotion ELN
  role: Main client receiving the composed peak table via the automated transfer of
    detected or manually chosen signals
  repo: https://github.com/ComPlat/chemotion_ELN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:49e0a345d130d87d95df6c71e2c8ad7d21c456bec4482bf3fd980d7da2d85d83
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# jcamp-peak-table-composition

## Summary

Composes an annotated JCAMP-DX peak table in which each detected spectral peak is written as its x-position (e.g. IR wavenumber) followed by a bracketed descriptor such as an intensity label (vw/w/m/s/vs for IR), reproducing the peak-table format ChemSpectra writes for transfer to the ELN or repository.

## When to use

Use when detected peak positions must be converted into an annotated peak table/label file (e.g. IR wavenumbers with bracketed vw/w/m/s/vs intensity labels) for export as JCAMP-DX metadata or ELN transfer.

## When NOT to use

- Input is a raw or proprietary file (e.g. MS RAW or NMR FID) that has not yet been converted to JCAMP-DX — convert first (RAW to mzML via msConvert, mzML to JCAMP-DX via pymzML) before composing the peak table
- The analysis goal is NMR-specific reporting such as integration, coupling constants or multiplicity assignment rather than peak-position/label annotation
- 2D NMR data — viewing and analysis of 2D data is not supported by ChemSpectra

## Inputs

- JCAMP-DX spectrum file (IR, MS, or 1D NMR) from which x/y data points have been extracted
- List of detected (or manually chosen) peak positions from automatic peak detection with a default or manually defined threshold
- Peak classification values, e.g. IR intensity category (vw, w, m, s, vs) per peak

## Outputs

- Annotated peak table mapping each peak position to a bracketed label (e.g. wavenumber + [w] for IR)
- Peak label export file (e.g. IR_peak_labels.txt) suitable for transfer to the Chemotion ELN or repository
- Updated JCAMP-DX output containing the composed peak table

## How to apply

First obtain the detected peak positions by applying ChemSpectra's automatic peak detection, which runs 'according to a default or manually defined threshold' on the x/y data extracted from the JCAMP-DX file (parsed e.g. via jcampconverter inside react-spectra-editor). Next, derive the annotation value for each peak: for IR spectra classify each detected peak into one of five intensity categories (very weak, weak, medium, strong, very strong) from its relative intensity within the spectrum. Then compose each annotation as the wavenumber with the intensity abbreviation placed 'in brackets after the corresponding wavenumber per IR reporting standards', e.g. [w]. Finally write the annotated peaks out — ChemSpectra implements 'a workflow to write the automatically detected or manually chosen signals', allowing direct transfer of the peak table to e.g. the Chemotion ELN or repository — and verify every detected peak carries exactly one valid bracketed label. Peaks may also come from manual selection rather than automatic detection; both paths feed the same composition step.

## Related tools

- **ChemSpectra** (Backend service performing automatic peak detection (default or manual threshold) and writing detected or manually chosen signals into the exportable peak table) — https://github.com/ComPlat/chem-spectra-app
- **react-spectra-editor** (JavaScript front end that extracts spectroscopic data from JCAMP-DX via jcampconverter and hosts the editor/control panel used to pick or adjust peaks before table composition) — https://github.com/ComPlat/react-spectra-editor
- **chem-spectra-client** (Standalone client that integrates react-spectra-editor and realizes peaks output and file management) — https://github.com/ComPlat/chem-spectra-client
- **Chemotion ELN** (Main client receiving the composed peak table via the automated transfer of detected or manually chosen signals) — https://github.com/ComPlat/chemotion_ELN

## Evaluation signals

- Every detected peak position in the output carries exactly one bracketed label, and for IR every label is one of the five intensity abbreviations vw, w, m, s, vs
- Each bracketed label appears after its corresponding wavenumber, matching IR reporting standards as implemented by ChemSpectra
- The set of annotated peaks matches the peak list produced by the threshold-based detection (default or manually defined) — no missing or extra peaks
- The composed peak table round-trips into a JCAMP-DX file that the react-spectra-editor/jcampconverter parser can re-read
- The exported peak information transfers cleanly into the Chemotion ELN via the implemented signal-transfer workflow

## Limitations

- ChemSpectra is a basic editor intended to be extended; further data type-specific analysis functions and additional file formats are stated as future work
- Only JCAMP-DX (IR, MS, NMR), mzML (MS) and extensions for proprietary RAW (MS) and FID (NMR) are handled; other formats must be converted to JCAMP-DX first
- Viewing and analysis of 2D NMR data and comparison of different spectra in one application window are not yet implemented

## Evidence

- [discussion] IR intensity annotation of detected peaks: [evidence span withheld — anchor status: not_found]
- [results] bracket placement per IR reporting standards: "the automatic peak detection according to a default or manually defined threshold"
- [discussion] signal writing workflow: "a workflow to write the automatically detected or manually chosen signals was implemented. This allows the direct transfer of information to e.g. the ELN or repository."
- [methods] JCAMP-DX parsing path: "The jcampconverter library [17] inside the react-spectra-editor extracts spectroscopic data of the composed JCAMP-DX file"
