---
name: jcamp-spectrum-parsing
description: Use when the input is a JCAMP-DX spectrum (or a file already converted
  to JCAMP-DX, e.g. mzML converted via pymzML) and the x/y data points must be extracted
  as the basis for peak picking, annotation, or peaks output.
when_to_use_negative:
- The input is a proprietary RAW (MS) or FID (NMR) file that must first be converted
  (RAW via msConvert to mzML, mzML via pymzML to JCAMP-DX) before JCAMP parsing applies.
- The analysis goal is 2D NMR viewing/analysis or spectra overlay comparison — both
  are explicitly listed as missing functionality in ChemSpectra.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3361
- http://edamontology.org/topic_0092
tools:
- name: ChemSpectra (chem-spectra-app)
  role: Backend that converts non-JCAMP-DX inputs to JCAMP-DX and decodes/peak-picks
    the parsed spectra files
  repo: https://github.com/ComPlat/chem-spectra-app
- name: react-spectra-editor
  role: Client-side editor whose jcampconverter library extracts spectroscopic data
    from the composed JCAMP-DX file
  repo: https://github.com/ComPlat/react-spectra-editor
- name: chem-spectra-client
  role: Standalone page calling the backend service to view/edit parsed spectra and
    produce peaks output
  repo: https://github.com/ComPlat/chem-spectra-client
- name: pymzML
  role: Open Source python mass spectrometry parser used to convert mzML to JCAMP-DX
    prior to parsing
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:7ab834d3be3acf59b82759dc298ec0db964c3e8d697f635703136c3db7aab43e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ComPlat/chem-spectra-app
  tools:
  - ChemSpectra
  - ChemSpectra (chem-spectra-app)
  - JavaScript
  - chem-spectra-client
  - pymzML
  - react-spectra-editor
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3361
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# jcamp-spectrum-parsing

## Summary

Parse JCAMP-DX spectroscopy files (IR, MS, 1H/13C NMR) into x/y data-point lists using the JCAMP reading approach of chem-spectra-app (nmrglue-based block reader) or the jcampconverter library inside react-spectra-editor, so downstream steps such as threshold-based peak detection operate on a standardized data table.

## When to use

Use when the input is a JCAMP-DX spectrum (or a file already converted to JCAMP-DX, e.g. mzML converted via pymzML) and the x/y data points must be extracted as the basis for peak picking, annotation, or peaks output.

## When NOT to use

- The input is a proprietary RAW (MS) or FID (NMR) file that must first be converted (RAW via msConvert to mzML, mzML via pymzML to JCAMP-DX) before JCAMP parsing applies.
- The analysis goal is 2D NMR viewing/analysis or spectra overlay comparison — both are explicitly listed as missing functionality in ChemSpectra.

## Inputs

- JCAMP-DX spectrum file (IR, MS, or 1H/13C NMR)
- mzML file (MS) already converted to JCAMP-DX via pymzML
- Bruker, BagIt, or RAW files after conversion to JCAMP-DX by the chem-spectra-app backend

## Outputs

- List of parsed data points as x/y pairs (e.g. wavenumber/intensity, chemical shift/intensity, m/z/intensity)
- Parsed JCAMP header/metadata blocks associated with the spectrum technique
- Structured data table fed into peak detection and the JCAMP peak table / peaks output

## How to apply

ChemSpectra standardizes all inputs by first converting non-JCAMP-DX files to JCAMP-DX (e.g. RAW MS to mzML with msConvert in a docker container, then mzML to JCAMP-DX with pymzML), allowing a uniform parsing step afterwards. Load the JCAMP-DX file with the JCAMP reader used by the chem-spectra-app converters (nmrglue-based block reader) or, on the client side, let the jcampconverter library inside react-spectra-editor extract the spectroscopic data of the composed JCAMP-DX file sent from the server. Parse the data table into ordered x/y pairs (wavenumber/intensity for IR, chemical shift/intensity for NMR, m/z/intensity for MS), which is the exact representation consumed by the threshold-based automatic peak detection. Retain the header/metadata blocks needed for the spectrum technique, since per-spectra-type defaults (e.g. the peak-detection threshold) are defined per technique. The parsed x/y list is then passed to peak extraction and written back into the JCAMP peak table format.

## Related tools

- **ChemSpectra (chem-spectra-app)** (Backend that converts non-JCAMP-DX inputs to JCAMP-DX and decodes/peak-picks the parsed spectra files) — https://github.com/ComPlat/chem-spectra-app
- **react-spectra-editor** (Client-side editor whose jcampconverter library extracts spectroscopic data from the composed JCAMP-DX file) — https://github.com/ComPlat/react-spectra-editor
- **chem-spectra-client** (Standalone page calling the backend service to view/edit parsed spectra and produce peaks output) — https://github.com/ComPlat/chem-spectra-client
- **pymzML** (Open Source python mass spectrometry parser used to convert mzML to JCAMP-DX prior to parsing)

## Examples

```
import nmrglue as ng; dic, data = ng.bruker.read_jcamp('input.jdx')
```

## Evaluation signals

- The parsed result is an ordered list of numeric x/y pairs covering the full spectral range of the input JCAMP-DX data table.
- Re-serializing the parsed points reproduces the original JCAMP-DX data table within floating-point tolerance.
- Downstream threshold-based peak detection (default or manually defined threshold) yields peaks whose positions fall within the parsed x range.
- The peaks.json output is valid JSON and each detected peak is a list of objects with a numeric x (and y) entry, matching the ChemSpectra peak structure.
- Non-JCAMP-DX inputs (RAW, mzML) were successfully converted to JCAMP-DX before parsing, per the standardized-treatment pipeline.

## Limitations

- Processing of FID files requires advanced phase correction and baseline correction methods, which are noted as improvements still to be added.
- The editor lacks functions for viewing/analysis of 2D NMR data and for comparing different spectra in one application window.
- Extension to further data-type-specific analysis functions and additional file formats is stated as future work.

## Evidence

- [methods] jcampconverter extracts data: "The jcampconverter library [17] inside the react-spectra-editor extracts spectroscopic data of the composed JCAMP-DX file which is sent from the server-side"
- [abstract] open file formats: "The software can deal with the open file formats JCAMP-DX (IR, MS, NMR) and mzML (MS) proposing these data file types to gain interoperable data"
- [discussion] conversion pipeline: "All data files that are provided as non-JCAMP-DX files are processed and converted to JCAMP-DX, allowing a standardized treatment of all data files after a first processing step."
- [readme] backend formats: "This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files."
