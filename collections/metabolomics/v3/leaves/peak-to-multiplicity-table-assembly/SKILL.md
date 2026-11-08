---
name: peak-to-multiplicity-table-assembly
description: Use when a 1H NMR spectrum (e.g. JCAMP-DX) has been loaded and peak-picked
  with a threshold, and you need per-peak coupling constants and multiplicity labels
  assembled into a structured CSV table.
when_to_use_negative:
- The input is 2D NMR data — the article lists viewing and analysis of 2D data as
  a future improvement not yet supported by ChemSpectra
- The input is a raw Bruker FID file requiring advanced phase correction and baseline
  correction — the article notes FID file processing still has to be improved
- The goal is IR or MS analysis — coupling constant calculation and multiplicity assignment
  are NMR-specific functions
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_0593
tools:
- name: ChemSpectra
  role: 'Backend service performing NMR-specific analysis: peak detection by threshold,
    coupling constant calculation and multiplicity assignment'
  repo: https://github.com/ComPlat/chem-spectra-app
- name: react-spectra-editor
  role: Client-side editor whose jcampconverter library extracts spectroscopic data
    from the JCAMP-DX file and displays/edits the 1H spectrum
  repo: https://github.com/ComPlat/react-spectra-editor
- name: nmrglue
  role: Python library used to read JCAMP-DX blocks and select the recognized 1H spectrum
    block
- name: SciPy
  role: Used for automatic peak detection on the spectrum trace according to the threshold
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:5322f6b592e4aa8490ad0e409368b6650d49b0e869f0ba7df47cc6ac6c405edd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ComPlat/chem-spectra-app
  tools:
  - ChemSpectra
  - Python
  - SciPy
  - nmrglue
  - react-spectra-editor
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_0593
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# peak-to-multiplicity-table-assembly

## Summary

Assembles a peak-to-multiplicity assignment table for a 1H NMR spectrum: picked peaks with positions (x) and intensities (y) are annotated with coupling constants computed from the splitting pattern of neighbouring lines and multiplicity labels (s, d, t, q, quint, m) inferred by known libraries and checked by additional rules, following the ChemSpectra workflow.

## When to use

Use when a 1H NMR spectrum (e.g. JCAMP-DX) has been loaded and peak-picked with a threshold, and you need per-peak coupling constants and multiplicity labels assembled into a structured CSV table.

## When NOT to use

- The input is 2D NMR data — the article lists viewing and analysis of 2D data as a future improvement not yet supported by ChemSpectra
- The input is a raw Bruker FID file requiring advanced phase correction and baseline correction — the article notes FID file processing still has to be improved
- The goal is IR or MS analysis — coupling constant calculation and multiplicity assignment are NMR-specific functions

## Inputs

- 1H NMR spectrum in JCAMP-DX format (converted via nmrglue read_blocks, 1H block selected via data_type.json)
- Peak-picking threshold (thres parameter, default or manually defined)
- Known libraries used to infer multiplicity assignments

## Outputs

- multiplicity_assignments.csv: one row per picked peak with fields peak position (x), intensity (y), coupling constant and multiplicity label
- Peak-to-multiplicity assignment table for transfer to an ELN or repository

## How to apply

Load the 1H NMR JCAMP-DX file with the jcamp converter (nmrglue read_blocks over the JCAMP blocks, selecting the recognized 1H spectrum block via the data_type.json mapping). Apply automatic peak detection to the trace using the default or a manually defined threshold (thres parameter) to obtain picked peaks with x/y positions. For each picked peak region in the 1H spectrum, compute the coupling constant(s) from the splitting pattern of neighbouring lines and assign the multiplicity label (e.g. s, d, t, q, m) according to the inferred number of coupling partners, using known libraries to infer assignments and then checking them with additional rules. Finally, assemble peak positions, intensities, coupling constants and multiplicity labels into a table and write it to multiplicity_assignments.csv.

## Related tools

- **ChemSpectra** (Backend service performing NMR-specific analysis: peak detection by threshold, coupling constant calculation and multiplicity assignment) — https://github.com/ComPlat/chem-spectra-app
- **react-spectra-editor** (Client-side editor whose jcampconverter library extracts spectroscopic data from the JCAMP-DX file and displays/edits the 1H spectrum) — https://github.com/ComPlat/react-spectra-editor
- **nmrglue** (Python library used to read JCAMP-DX blocks and select the recognized 1H spectrum block)
- **SciPy** (Used for automatic peak detection on the spectrum trace according to the threshold)

## Examples

```
from scipy.signal import find_peaks; peaks, props = find_peaks(y, height=thres); # then compute J from split-line spacing and write multiplicity_assignments.csv
```

## Evaluation signals

- multiplicity_assignments.csv exists and is valid CSV
- The CSV contains one row per picked peak, matching the number of peaks detected at the chosen threshold
- Each row has fields for peak position (x), intensity (y), coupling constant and multiplicity
- All multiplicity labels are drawn from the standard set {s, d, t, q, quint, m}
- Inferred multiplicity assignments are consistent with the splitting pattern of neighbouring lines (checked by the additional rules applied after library-based inference)

## Limitations

- Viewing and analysis of 2D NMR data is not yet supported and is listed as a future improvement
- Processing of FID files has to be improved by adding advanced phase correction and baseline correction methods
- No quantitative evaluation, performance metrics, or user study results are reported for ChemSpectra in the article
- Multiplicity inference depends on known libraries and additional checking rules; heavily overlapping or unusual splitting may fall back to 'm'

## Evidence

- [abstract] NMR-specific functions include reference signal definition, coupling constant calculation and multiplicity assignment: "NMR specific functions include the definition of a reference signal, the integration of signals, coupling constant calculation and multiplicity assignment"
- [discussion] Automatic peak detection threshold drives the picked peaks that feed the multiplicity table: "the automatic peak detection according to a default or manually defined threshold"
- [methods] jcampconverter extracts the spectroscopic data from the JCAMP-DX file before analysis: "The jcampconverter library [17] inside the react-spectra-editor extracts spectroscopic data of the composed JCAMP-DX file which is sent from the server-side"
- [readme] ChemSpectra backend accepts 1H NMR JCAMP-DX as input: "This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files."
