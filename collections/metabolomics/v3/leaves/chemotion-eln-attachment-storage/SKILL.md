---
name: chemotion-eln-attachment-storage
description: 'Use when a host application or pipeline must store processed spectra
  per the Chemotion ELN contract: one original byte-preserving artifact plus a server-composed
  edited version and generated png per upload.'
when_to_use_negative:
- The goal is interactive spectrum editing or NMR analysis (peak-picking, integration,
  coupling constants) rather than per-spectrum attachment storage
- The input is a proprietary MS RAW file that first requires msConvert-in-docker conversion
  to mzML and then JCAMP-DX via pymzML before storage
- The data is 2D NMR or formats outside JCAMP-DX/mzML/Bruker/BagIt scope, which chem-spectra-app
  does not cover per the paper's stated future work
edam_topics:
- http://edamontology.org/topic_3315
tools:
- name: ChemSpectra
  role: Backend service whose /jcamp and /image endpoints compose the edited JCAMP-DX
    version and the generated png per uploaded spectrum
  repo: https://github.com/ComPlat/chem-spectra-app
- name: Chemotion ELN
  role: Host application that defines the three-file-per-spectrum attachment storage
    contract (original, last edited version, generated png)
  repo: https://github.com/ComPlat/chemotion_ELN
- name: react-spectra-editor
  role: Client whose jcampconverter library parses the composed JCAMP-DX file, used
    to validate the edited output artifact
  repo: https://github.com/ComPlat/react-spectra-editor
- name: JavaScript
  role: Implementation language of the editor that can be integrated with other software
    for spectrum file management
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:92d4499c1339bd81d94cba073ba59f64e69de5183b2e1d093524a1e6ce34a39e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ComPlat/chem-spectra-app
  tools:
  - ChemSpectra
  - Chemotion ELN
  - JavaScript
  - react-spectra-editor
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3315
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# chemotion-eln-attachment-storage

## Summary

Replicates the file-management contract that the Chemotion ELN implements around a single uploaded spectrum by producing three artifacts per file: the untouched original upload, the last edited JCAMP-DX version, and an automatically generated png image of the spectra.

## When to use

Use when a host application or pipeline must store processed spectra per the Chemotion ELN contract: one original byte-preserving artifact plus a server-composed edited version and generated png per upload.

## When NOT to use

- The goal is interactive spectrum editing or NMR analysis (peak-picking, integration, coupling constants) rather than per-spectrum attachment storage
- The input is a proprietary MS RAW file that first requires msConvert-in-docker conversion to mzML and then JCAMP-DX via pymzML before storage
- The data is 2D NMR or formats outside JCAMP-DX/mzML/Bruker/BagIt scope, which chem-spectra-app does not cover per the paper's stated future work

## Inputs

- A single uploaded spectrum file in JCAMP-DX format (.jdx, .dx, .jcamp) for IR, MS, or 1D 1H/13C NMR, or a Bruker zip archive
- Optionally mzML (MS) or proprietary RAW/FID files after prior conversion to JCAMP-DX via msConvert/pymzML

## Outputs

- An original file (*.zip) byte-identical to the upload
- A user-edited spectrum version (*.edit.jdx) produced by the /jcamp transformation endpoint
- An automatically generated png image (*.edit.png) produced by the /image endpoint

## How to apply

Read the uploaded spectrum file (JCAMP-DX .jdx/.dx/.jcamp or Bruker zip) as raw bytes so the original artifact remains unmodified, mirroring chem-spectra-app's FileContainer behavior. Post the spectrum to the chem-spectra-app backend (a Flask service built on nmrglue and SciPy) via the /jcamp transformation endpoint to obtain the server-composed edited JCAMP-DX version, then post the same file to the /image endpoint so the matplotlib-based composer renders the png. Write the untouched original bytes unchanged, save the returned spectrum.jdx as the last-edited version (*.edit.jdx), and save spectrum.png (*.edit.png). The result mirrors the three files the Chemotion ELN stores per spectrum, as described in the paper's file-management finding.

## Related tools

- **ChemSpectra** (Backend service whose /jcamp and /image endpoints compose the edited JCAMP-DX version and the generated png per uploaded spectrum) — https://github.com/ComPlat/chem-spectra-app
- **Chemotion ELN** (Host application that defines the three-file-per-spectrum attachment storage contract (original, last edited version, generated png)) — https://github.com/ComPlat/chemotion_ELN
- **react-spectra-editor** (Client whose jcampconverter library parses the composed JCAMP-DX file, used to validate the edited output artifact) — https://github.com/ComPlat/react-spectra-editor
- **JavaScript** (Implementation language of the editor that can be integrated with other software for spectrum file management)

## Examples

```
curl -F 'file=@spectrum.dx' http://localhost:8080/api/v1/jcamp -o spectrum.edit.jdx && curl -F 'file=@spectrum.dx' http://localhost:8080/api/v1/image -o spectrum.edit.png
```

## Evaluation signals

- The output directory contains exactly three files: the original upload, the edited JCAMP-DX version, and the png image
- The stored original file is byte-identical to the uploaded spectrum (checksum comparison)
- The *.edit.jdx output can be parsed back into spectroscopic data by the jcampconverter library used inside react-spectra-editor
- The *.edit.png is a non-empty, renderable image of the spectra
- File naming follows the ELN convention: original (*.zip), edited (*.edit.jdx), image (*.edit.png)

## Limitations

- 2D NMR viewing/analysis and spectra comparison in one window are not supported and listed as future improvements
- FID file processing lacks advanced phase and baseline correction methods
- No quantitative evaluation, performance metrics, or user study results are reported for this file-management workflow
- The paper does not specify version numbers or release details for the ChemSpectra modules

## Evidence

- [abstract] File management stores original spectra, the last edited version, and an automatically generated png image: "The file management supports the storage of the original spectra along with the last edited version and an automatically generated image of the spectra in png format"
- [methods] Edited spectra are composed server-side and parsed back by jcampconverter: "The jcampconverter library [17] inside the react-spectra-editor extracts spectroscopic data of the composed JCAMP-DX file which is sent from the server-side"
- [discussion] Non-JCAMP-DX inputs are standardized to JCAMP-DX before storage: "All data files that are provided as non-JCAMP-DX files are processed and converted to JCAMP-DX, allowing a standardized treatment of all data files after a first processing step."
- [readme] chem-spectra-client realizes the file management for spectra editing: [evidence span withheld — anchor status: external_doc]
