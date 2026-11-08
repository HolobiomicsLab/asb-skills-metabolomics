---
name: jcamp-file-conversion
description: Use when input spectra are in mzML, proprietary RAW/FID, Bruker zip,
  or other non-JCAMP-DX formats and must be normalized to open JCAMP-DX (IR, MS, 1H/13C
  NMR) for ELN storage, editing, or image generation.
when_to_use_negative:
- Input is already an open JCAMP-DX or mzML file and the goal is only visualization/peak-picking,
  not format normalization
- The data are 2D NMR spectra — viewing and analysis of 2D data are explicitly listed
  as missing functionality to be added in future work
- A quick desktop analysis is sufficient and ELN/repository integration with file
  management (original + edited + png) is not required
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: ChemSpectra (chem-spectra-app)
  role: Flask backend performing the JCAMP-DX conversion pipeline via TransformerModel,
    JcampBaseConverter, and TechniqueComposer/MSComposer
  repo: https://github.com/ComPlat/chem-spectra-app
- name: react-spectra-editor
  role: 'Client-side validation layer: its jcampconverter library parses the composed
    JCAMP-DX returned by the server'
  repo: https://github.com/ComPlat/react-spectra-editor
- name: pymzML
  role: Open Source python parser used to convert mzML files to JCAMP-DX
- name: MsConvert (ProteoWizard)
  role: Called in a docker container by chem-spectra-app to decode proprietary RAW
    MS files into mzML before JCAMP-DX conversion
- name: Chemotion ELN
  role: Main client that embeds the service and stores original, last-edited, and
    generated png files per spectrum
  repo: https://github.com/ComPlat/chemotion_ELN
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:a8e43fed36fe6f53d1cf19dce93fab9397a7ec87020d8db58827b5d992859fb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ComPlat/chem-spectra-app
  tools:
  - ChemSpectra
  - ChemSpectra (chem-spectra-app)
  - Chemotion ELN
  - JavaScript
  - MsConvert (ProteoWizard)
  - pymzML
  - react-spectra-editor
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# jcamp-file-conversion

## Summary

Convert heterogeneous spectroscopy inputs (mzML, proprietary MS RAW, Bruker/FID, non-JCAMP files) into standardized JCAMP-DX using ChemSpectra's chem-spectra-app conversion pipeline, so all data types get uniform downstream handling, editing, and png-image generation.

## When to use

Use when input spectra are in mzML, proprietary RAW/FID, Bruker zip, or other non-JCAMP-DX formats and must be normalized to open JCAMP-DX (IR, MS, 1H/13C NMR) for ELN storage, editing, or image generation.

## When NOT to use

- Input is already an open JCAMP-DX or mzML file and the goal is only visualization/peak-picking, not format normalization
- The data are 2D NMR spectra — viewing and analysis of 2D data are explicitly listed as missing functionality to be added in future work
- A quick desktop analysis is sufficient and ELN/repository integration with file management (original + edited + png) is not required

## Inputs

- JCAMP-DX spectra files (.jdx/.dx/.jcamp) for IR, MS, or 1D 1H/13C NMR
- mzML files (MS)
- Proprietary RAW mass spectrometry files
- Bruker zip archives / NMR FID files
- Form parameters (e.g. params['ext']) sent with the upload to the transform endpoint

## Outputs

- Composed JCAMP-DX file (e.g. spectrum.jdx / *.edit.jdx) parseable by the jcampconverter library
- Automatically generated png image of the spectra (spectrum.png / *.edit.png) rendered at 200 dpi via the matplotlib-based composer
- Unmodified original artifact (e.g. original *.zip) stored alongside the edited and image files

## How to apply

Route the upload through chem-spectra-app: the controller wraps the file in a FileContainer and passes it with the extracted 'ext' parameter to TransformerModel, whose to_composer() picks the conversion route from the file extension. For MS data, first decode proprietary RAW files by calling MsConvert (ProteoWizard) in a docker container to obtain mzML, then convert mzML to JCAMP-DX with pymzML. For JCAMP-like input (e.g. .jdx/.dx/.jcamp), route through jcamp2cvp() to a JcampBaseConverter and then a TechniqueComposer (MSComposer for MS data). The rationale is that all non-JCAMP-DX files are converted to JCAMP-DX as a first processing step, giving a standardized treatment of all data files; the composed JCAMP-DX is parsed on the client by the jcampconverter library inside react-spectra-editor, so validate round-trip parseability.

## Related tools

- **ChemSpectra (chem-spectra-app)** (Flask backend performing the JCAMP-DX conversion pipeline via TransformerModel, JcampBaseConverter, and TechniqueComposer/MSComposer) — https://github.com/ComPlat/chem-spectra-app
- **react-spectra-editor** (Client-side validation layer: its jcampconverter library parses the composed JCAMP-DX returned by the server) — https://github.com/ComPlat/react-spectra-editor
- **pymzML** (Open Source python parser used to convert mzML files to JCAMP-DX)
- **MsConvert (ProteoWizard)** (Called in a docker container by chem-spectra-app to decode proprietary RAW MS files into mzML before JCAMP-DX conversion)
- **Chemotion ELN** (Main client that embeds the service and stores original, last-edited, and generated png files per spectrum) — https://github.com/ComPlat/chemotion_ELN

## Examples

```
curl -X POST -F "file=sample.jdx" -F "ext=jdx" http://localhost:4000/jcamp
```

## Evaluation signals

- The output directory contains exactly three files per uploaded spectrum: an untouched original (*.zip), a user-edited version (*.edit.jdx), and an image file (*.edit.png)
- The original output file is byte-identical to the upload
- The composed JCAMP-DX edited version can be parsed back into spectroscopic data by the jcampconverter library
- spectrum.png exists, is non-empty, and is a valid PNG rendered from the uploaded spectrum data
- In the ELN flow, both low-resolution thumbnail and higher-resolution png are regenerated after every spectrum edit

## Limitations

- Proprietary RAW (MS) and FID (NMR) support is an extension exemplified for selected files; RAW decoding depends on MsConvert running in a docker container
- Editing 2D NMR data and comparing different spectra in one window are not yet supported
- FID file processing needs improved phase and baseline correction methods; further MS file formats are planned future extensions

## Evidence

- [abstract] open formats: "The software can deal with the open file formats JCAMP-DX (IR, MS, NMR) and mzML (MS) proposing these data file types to gain interoperable data"
- [results] pymzML conversion: "mzML files are converted to JCAMP-DX using pymzML, an Open Source python mass spectrometry file parser."
- [results] MsConvert docker: "MsConvert in a docker container is called by the chem-spectra-app to achieve this job."
- [discussion] standardized first step: "All data files that are provided as non-JCAMP-DX files are processed and converted to JCAMP-DX, allowing a standardized treatment of all data files after a first processing step."
- [methods] client-side parse: "The jcampconverter library [17] inside the react-spectra-editor extracts spectroscopic data of the composed JCAMP-DX file which is sent from the server-side"
- [readme] README scope: "This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files."
