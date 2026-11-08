---
name: nmr-jcamp-dx-block-parsing
description: Use when a 1H or 13C NMR input file is in JCAMP-DX (.jdx) format and
  the spectrum must be loaded block-by-block before peak detection or multiplicity
  analysis, as done in ChemSpectra's NMR workflow.
when_to_use_negative:
- Input is already parsed peak data (peak x/y positions with multiplicities) — go
  straight to multiplicity assignment instead
- Input is 2D NMR data — ChemSpectra does not yet support viewing/analysis of 2D data
- Input is proprietary Bruker FID or MS RAW/mzML — convert to JCAMP-DX first (msConvert/pymzML)
  before block parsing
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3314
tools:
- name: nmrglue
  role: Python library used to read the JCAMP-DX blocks of the NMR file (read_blocks)
    and expose the spectrum trace
- name: ChemSpectra (chem-spectra-app)
  role: Backend service whose jcamp converter parses JCAMP-DX NMR files and maps recognized
    data types via data_type.json
  repo: https://github.com/ComPlat/chem-spectra-app
- name: react-spectra-editor
  role: Frontend whose jcampconverter library extracts the spectroscopic data of the
    composed JCAMP-DX file sent from the server side
  repo: https://github.com/ComPlat/react-spectra-editor
- name: SciPy
  role: Python numerics used in the peak-picking/analysis stage after the block is
    parsed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:3a4ec062dd14a5e2d60ea96a6ec543a030ad73bc0276b4b52073dbde145e4ac5
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
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# nmr-jcamp-dx-block-parsing

## Summary

Parse JCAMP-DX NMR files by iterating over their data blocks (nmrglue read_blocks) and selecting the recognized 1H or 13C block via a data_type.json mapping, so downstream peak-picking, coupling-constant calculation and multiplicity assignment operate on the correct spectrum.

## When to use

Use when a 1H or 13C NMR input file is in JCAMP-DX (.jdx) format and the spectrum must be loaded block-by-block before peak detection or multiplicity analysis, as done in ChemSpectra's NMR workflow.

## When NOT to use

- Input is already parsed peak data (peak x/y positions with multiplicities) — go straight to multiplicity assignment instead
- Input is 2D NMR data — ChemSpectra does not yet support viewing/analysis of 2D data
- Input is proprietary Bruker FID or MS RAW/mzML — convert to JCAMP-DX first (msConvert/pymzML) before block parsing

## Inputs

- 1H NMR spectrum in JCAMP-DX (.jdx) format (possibly multi-block)
- 13C NMR spectrum in JCAMP-DX format
- data_type.json mapping of recognized JCAMP data types

## Outputs

- Parsed spectrum trace (x/y intensity arrays) for the recognized NMR block
- Block dictionary with recognized data_type label (e.g. 1H NMR)
- Parsed peak list feeding downstream multiplicity assignment

## How to apply

Load the 1H (or 13C) NMR JCAMP-DX file with ChemSpectra's jcamp converter: use nmrglue's read_blocks over the JCAMP blocks, and select the recognized NMR spectrum block via the data_type.json mapping rather than assuming the first block is the spectrum. Rationale: JCAMP-DX files can be composed of multiple blocks, and ChemSpectra standardizes all data to JCAMP-DX as a first processing step, so a recognized-type lookup is the robust way to pick the right block. Once the correct 1H block is selected, expose its trace (x/y arrays) for automatic peak detection using a default or manually defined threshold (thres parameter), which feeds the subsequent multiplicity/coupling-constant assignment. If the input is non-JCAMP-DX (e.g. Bruker FID or RAW/mzML for MS), convert to JCAMP-DX first (msConvert/pymzML on the ChemSpectra side) before applying this block-parsing skill.

## Related tools

- **nmrglue** (Python library used to read the JCAMP-DX blocks of the NMR file (read_blocks) and expose the spectrum trace)
- **ChemSpectra (chem-spectra-app)** (Backend service whose jcamp converter parses JCAMP-DX NMR files and maps recognized data types via data_type.json) — https://github.com/ComPlat/chem-spectra-app
- **react-spectra-editor** (Frontend whose jcampconverter library extracts the spectroscopic data of the composed JCAMP-DX file sent from the server side) — https://github.com/ComPlat/react-spectra-editor
- **SciPy** (Python numerics used in the peak-picking/analysis stage after the block is parsed)

## Examples

```
import nmrglue as ng; dic, data = ng.jcamp.read('1H_spectrum.jdx')  # select the recognized 1H block via data_type.json before peak picking
```

## Evaluation signals

- The parsed block's recognized data_type label matches a 1H (or 13C) NMR entry in data_type.json, not an IR/MS type
- The parsed trace has monotonic x values and y intensities consistent with the JCAMP-DX ##XYDATA block
- Subsequent automatic peak detection with the default or manually defined threshold (thres) yields picked peaks with valid x/y positions
- The output assignment table contains one row per picked peak with position (x), intensity (y), coupling constant and multiplicity from {s, d, t, q, quint, m}

## Limitations

- ChemSpectra handles one-dimensional 1H and 13C NMR only; 2D NMR viewing/analysis is listed as future work
- Processing of FID files needs improvement (advanced phase/baseline correction methods are not yet implemented)
- Non-JCAMP-DX files must be converted to JCAMP-DX in a first processing step; RAW (MS) requires msConvert in a docker container

## Evidence

- [abstract] abstract: "The software can deal with the open file formats JCAMP-DX (IR, MS, NMR) and mzML (MS) proposing these data file types to gain interoperable data"
- [methods] methods: "The jcampconverter library [17] inside the react-spectra-editor extracts spectroscopic data of the composed JCAMP-DX file which is sent from the server-side"
- [discussion] discussion: "All data files that are provided as non-JCAMP-DX files are processed and converted to JCAMP-DX, allowing a standardized treatment of all data files after a first processing step."
- [readme] readme: "This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files."
