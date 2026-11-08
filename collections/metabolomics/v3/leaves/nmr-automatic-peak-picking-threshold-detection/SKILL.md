---
name: nmr-automatic-peak-picking-threshold-detection
description: Use when a 1D 1H or 13C NMR spectrum (JCAMP-DX) must be reduced to a
  picked peak list — e.g. as the first step before integration, coupling constant
  calculation, multiplicity assignment, or automated transfer of signals to an ELN.
when_to_use_negative:
- Input is 2D NMR data — viewing and analysis of 2D data is explicitly listed as a
  missing signal/future improvement in ChemSpectra
- Input is a raw NMR FID requiring advanced phase or baseline correction — the article
  notes FID file processing still needs these improved methods
- The goal is generic IR or MS peak analysis rather than NMR-specific multiplicity/coupling
  work — this skill is scoped to 1D 1H/13C NMR peak picking
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: ChemSpectra (chem-spectra-app)
  role: Backend web service that loads JCAMP-DX NMR files and performs automatic peak
    detection by threshold, plus NMR-specific coupling constant calculation and multiplicity
    assignment
  repo: https://github.com/ComPlat/chem-spectra-app
- name: react-spectra-editor
  role: Frontend whose jcampconverter library extracts the spectroscopic data of the
    JCAMP-DX file sent from the server, displaying/editing picked peaks
  repo: https://github.com/ComPlat/react-spectra-editor
- name: nmrglue
  role: Used with Python/SciPy to read the JCAMP blocks and select the recognized
    1H spectrum block during the NMR analysis workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:49b0e429b88cd1bd75fcc99c629645a232a997fc82190afd406604e17c67849f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ComPlat/chem-spectra-app
  tools:
  - ChemSpectra
  - ChemSpectra (chem-spectra-app)
  - Python
  - SciPy
  - nmrglue
  - react-spectra-editor
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# nmr-automatic-peak-picking-threshold-detection

## Summary

Automatic peak detection on 1D 1H/13C NMR traces using a default or manually defined intensity threshold, as implemented in ChemSpectra (chem-spectra-app + react-spectra-editor), feeding downstream coupling constant calculation, multiplicity assignment, and signal transfer to the Chemotion ELN.

## When to use

Use when a 1D 1H or 13C NMR spectrum (JCAMP-DX) must be reduced to a picked peak list — e.g. as the first step before integration, coupling constant calculation, multiplicity assignment, or automated transfer of signals to an ELN.

## When NOT to use

- Input is 2D NMR data — viewing and analysis of 2D data is explicitly listed as a missing signal/future improvement in ChemSpectra
- Input is a raw NMR FID requiring advanced phase or baseline correction — the article notes FID file processing still needs these improved methods
- The goal is generic IR or MS peak analysis rather than NMR-specific multiplicity/coupling work — this skill is scoped to 1D 1H/13C NMR peak picking

## Inputs

- 1D 1H or 13C NMR spectrum in JCAMP-DX format (proprietary FID/RAW inputs are first converted to JCAMP-DX by the service)
- Optional peak-picking threshold (thres) — default or manually defined

## Outputs

- Picked peak list with x (position) and y (intensity) values
- Coupling constants computed from splitting patterns of neighbouring lines
- Multiplicity labels per peak from {s, d, t, q, quint, m}
- Peak-to-multiplicity assignment table (multiplicity_assignments.csv)
- Automatically detected or manually chosen signals written for transfer to the Chemotion ELN/repository

## How to apply

Load the NMR spectrum with chem-spectra-app's jcamp converter (nmrglue read_blocks over the JCAMP blocks, selecting the recognized 1H block via data_type.json mapping); non-JCAMP-DX inputs are first converted to JCAMP-DX so all files get a standardized treatment. Apply automatic peak detection to the spectrum trace according to a default or manually defined threshold (thres parameter) to obtain picked peaks with x/y positions. For 1H NMR, multiplicities are automatically inferred by known libraries and then checked by additional rules to ensure correctness; compute coupling constants from the splitting pattern of neighbouring lines and label multiplicities from the standard set {s, d, t, q, quint, m}. Assemble peak positions, intensities, coupling constants and multiplicity labels into a peak-to-multiplicity assignment table (e.g. multiplicity_assignments.csv), which can be written out for direct transfer to the Chemotion ELN or repository.

## Related tools

- **ChemSpectra (chem-spectra-app)** (Backend web service that loads JCAMP-DX NMR files and performs automatic peak detection by threshold, plus NMR-specific coupling constant calculation and multiplicity assignment) — https://github.com/ComPlat/chem-spectra-app
- **react-spectra-editor** (Frontend whose jcampconverter library extracts the spectroscopic data of the JCAMP-DX file sent from the server, displaying/editing picked peaks) — https://github.com/ComPlat/react-spectra-editor
- **nmrglue** (Used with Python/SciPy to read the JCAMP blocks and select the recognized 1H spectrum block during the NMR analysis workflow)

## Examples

```
from scipy.signal import find_peaks; peaks, props = find_peaks(trace, height=thres)  # thres = default or manually defined NMR peak-picking threshold
```

## Evaluation signals

- multiplicity_assignments.csv exists, is valid CSV, and contains one row per picked peak
- Each row carries fields for peak position (x), intensity (y), coupling constant, and multiplicity
- All multiplicity labels are drawn from the standard set {s, d, t, q, quint, m}
- Every reported peak exceeds the configured threshold (default or manually defined thres) in the spectrum trace
- Inferred multiplicity assignments pass the additional rule-based checks ChemSpectra applies after library-based inference

## Limitations

- Only one-dimensional 1H and 13C NMR is covered; viewing and analysis of 2D NMR data is a stated future improvement
- Processing of FID files requires improved advanced phase correction and baseline correction methods, which are not yet available
- Multiplicities are inferred from known libraries and must be validated by additional rules; the paper reports no quantitative evaluation or performance metrics for the automatic assignment
- ChemSpectra is positioned as a basic software to be extended with further data-type-specific analysis functions and additional file formats

## Evidence

- [discussion] Automatic peak detection according to a default or manually defined threshold: "the automatic peak detection according to a default or manually defined threshold"
- [abstract] NMR-specific functions include coupling constant calculation and multiplicity assignment: "NMR specific functions include the definition of a reference signal, the integration of signals, coupling constant calculation and multiplicity assignment"
- [methods] jcampconverter extracts JCAMP-DX data in react-spectra-editor: "The jcampconverter library [17] inside the react-spectra-editor extracts spectroscopic data of the composed JCAMP-DX file which is sent from the server-side"
- [discussion] Detected signals are written for transfer to the ELN or repository: "a workflow to write the automatically detected or manually chosen signals was implemented. This allows the direct transfer of information to e.g. the ELN or repository."
- [readme] ChemSpectra backend processes NMR spectra from JCAMP-DX and other formats: "This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files."
- [results] 2D NMR analysis is a missing capability: "further improvements of the editor should contain functions for the viewing and analysis of 2D data"
