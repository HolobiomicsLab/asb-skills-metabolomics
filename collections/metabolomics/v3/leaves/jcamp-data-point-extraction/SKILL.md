---
name: jcamp-data-point-extraction
description: Use when the input is a JCAMP-DX (or mzML-converted-to-JCAMP-DX) spectrum
  file whose data table of x/y pairs must be parsed out before peak-picking, plotting,
  or writing a peaks list.
when_to_use_negative:
- The input is already a parsed data-point array or a peaks list rather than a JCAMP-DX/mzML
  spectrum file
- The task is 2D NMR viewing/analysis or FID phase/baseline correction — the article
  lists these as missing signals not covered by ChemSpectra
- The spectrum is in a proprietary format that is not one of the supported extension
  paths (RAW, FID, Bruker, BagIt) and no conversion route is available
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: ChemSpectra
  role: Web-based service whose chem-spectra-app backend decodes JCAMP-DX spectra
    files and parses data tables for downstream peak-picking
  repo: github.com/ComPlat/chem-spectra-app
- name: JavaScript
  role: Implementation language of the client-side editor in which the jcampconverter
    library extracts spectroscopic data from the composed JCAMP-DX file
  repo: https://github.com/ComPlat/react-spectra-editor
- name: react-spectra-editor
  role: Hosts the jcampconverter library that extracts spectroscopic data of the composed
    JCAMP-DX file on the client side
  repo: https://github.com/ComPlat/react-spectra-editor
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:27126eb52ae450b2596ddb607e77e1b682adbb67f5bcffdb3f89b4fe24525399
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemSpectra
  - JavaScript
  - react-spectra-editor
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# jcamp-data-point-extraction

## Summary

Extracts the x/y data-point table from JCAMP-DX spectroscopy files (IR, MS, 1D 1H/13C NMR) using the ChemSpectra/chem-spectra-app JCAMP reading approach, providing the parsed data points that downstream steps such as threshold-based peak detection operate on.

## When to use

Use when the input is a JCAMP-DX (or mzML-converted-to-JCAMP-DX) spectrum file whose data table of x/y pairs must be parsed out before peak-picking, plotting, or writing a peaks list.

## When NOT to use

- The input is already a parsed data-point array or a peaks list rather than a JCAMP-DX/mzML spectrum file
- The task is 2D NMR viewing/analysis or FID phase/baseline correction — the article lists these as missing signals not covered by ChemSpectra
- The spectrum is in a proprietary format that is not one of the supported extension paths (RAW, FID, Bruker, BagIt) and no conversion route is available

## Inputs

- JCAMP-DX spectrum file (IR, MS, or 1D 1H/13C NMR)
- mzML file (converted to JCAMP-DX with pymzML before parsing)
- Proprietary MS RAW file (converted to mzML via MsConvert/ProteoWizard docker container first)
- NMR FID file (proprietary format, extension path)

## Outputs

- Parsed list of data points as x/y pairs from the JCAMP-DX data table
- Structured JCAMP-DX representation usable by MSComposer/TechniqueComposer peak extraction

## How to apply

Load the input JCAMP-DX file and parse its data table to obtain the list of data points (x/y pairs), following the JCAMP reading approach of the chem-spectra-app converters, which use an nmrglue-based block reader on the backend; on the client side, the jcampconverter library inside react-spectra-editor extracts spectroscopic data from the composed JCAMP-DX file sent by the server. If the file is not already JCAMP-DX, first convert it: mzML files are converted to JCAMP-DX with pymzML, and proprietary MS RAW files are converted to mzML by MsConvert (ProteoWizard) in a docker container called by chem-spectra-app, so that all data receive standardized treatment after this first processing step. The parsed x/y pairs then feed automatic peak detection relative to a per-spectra-type default or manually defined threshold. Validate the parse by checking that the extracted data table yields numeric x/y pairs consistent with the spectrum's technique (IR, MS, NMR).

## Related tools

- **ChemSpectra** (Web-based service whose chem-spectra-app backend decodes JCAMP-DX spectra files and parses data tables for downstream peak-picking) — github.com/ComPlat/chem-spectra-app
- **JavaScript** (Implementation language of the client-side editor in which the jcampconverter library extracts spectroscopic data from the composed JCAMP-DX file) — https://github.com/ComPlat/react-spectra-editor
- **react-spectra-editor** (Hosts the jcampconverter library that extracts spectroscopic data of the composed JCAMP-DX file on the client side) — https://github.com/ComPlat/react-spectra-editor

## Evaluation signals

- The parsed data table yields numeric x/y pairs for every point in the spectrum
- Non-JCAMP-DX inputs (mzML, RAW) have been converted to JCAMP-DX before parsing, per the standardized first processing step
- The parsed points reproduce the spectrum when plotted, consistent with react-spectra-editor visualization
- Downstream threshold-based peak detection over the parsed points returns a peaks list whose x entries fall within the spectrum's x range

## Limitations

- Non-JCAMP-DX formats require a conversion step (pymzML for mzML; MsConvert docker call for RAW) before parsing
- 2D NMR data viewing/analysis is not supported; planned FID processing improvements (advanced phase and baseline correction) mean raw FID-derived JCAMP may need extra care
- Extension to additional file formats and data-type-specific analysis functions is stated as future work

## Evidence

- [methods] jcampconverter extracts data: "The jcampconverter library [17] inside the react-spectra-editor extracts spectroscopic data of the composed JCAMP-DX file which is sent from the server-side"
- [discussion] standardized JCAMP treatment: "All data files that are provided as non-JCAMP-DX files are processed and converted to JCAMP-DX, allowing a standardized treatment of all data files after a first processing step."
- [abstract] supported formats: "The software can deal with the open file formats JCAMP-DX (IR, MS, NMR) and mzML (MS) proposing these data file types to gain interoperable data"
- [readme] backend scope: "This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files."
