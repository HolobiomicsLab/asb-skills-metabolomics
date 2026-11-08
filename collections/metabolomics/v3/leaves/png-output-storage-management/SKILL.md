---
name: png-output-storage-management
description: Use when a web service or ELN must auto-generate a spectrum image (e.g.
  spectrum.png) from an uploaded JCAMP-DX/mzML spectrum and store it together with
  the original and edited spectral files, regenerating on edits.
when_to_use_negative:
- The user only needs interactive visualization/editing — that is handled by react-spectra-editor
  in the client, not by PNG generation
- The input is a proprietary format that cannot first be converted to JCAMP-DX (e.g.
  RAW needs msConvert, FID processing is flagged as an improvement area)
- 2D NMR data is required — viewing and analysis of 2D data is listed as missing functionality
edam_operation: http://edamontology.org/operation_3200
tools:
- name: ChemSpectra
  role: Backend service whose /image endpoint wraps the upload in a FileContainer,
    runs TransformerModel/TechniqueComposer, and returns the rendered spectrum.png
    via send_file
  repo: https://github.com/ComPlat/chem-spectra-app
- name: Chemotion ELN
  role: Main client that embeds ChemSpectra and stores the thumbnail and high-resolution
    PNG alongside the original and edited spectra
  repo: https://github.com/ComPlat/chemotion_ELN
- name: react-spectra-editor
  role: Front-end editor that displays the edited spectra whose last state the stored
    PNG must mirror
  repo: https://github.com/ComPlat/react-spectra-editor
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:4222a31fc9cd4c32f00ff89e5d1648c8c60aafc8dd60db6d0f5334e0936796b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ComPlat/chem-spectra-app
  tools:
  - ChemSpectra
  - Chemotion ELN
  - react-spectra-editor
  edam_operation: http://edamontology.org/operation_3200
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# png-output-storage-management

## Summary

Generates and manages automatically rendered PNG images of spectra (a preview thumbnail plus a higher-resolution reusable image) as part of the ChemSpectra file-management model, alongside the original and last-edited spectra files.

## When to use

Use when a web service or ELN must auto-generate a spectrum image (e.g. spectrum.png) from an uploaded JCAMP-DX/mzML spectrum and store it together with the original and edited spectral files, regenerating on edits.

## When NOT to use

- The user only needs interactive visualization/editing — that is handled by react-spectra-editor in the client, not by PNG generation
- The input is a proprietary format that cannot first be converted to JCAMP-DX (e.g. RAW needs msConvert, FID processing is flagged as an improvement area)
- 2D NMR data is required — viewing and analysis of 2D data is listed as missing functionality

## Inputs

- Uploaded spectrum file in JCAMP-DX format (.jdx/.dx/.jcamp) for IR, 1H/13C NMR, or MS
- mzML file (converted from proprietary RAW via msConvert, or via pymzML to JCAMP-DX)
- Form parameters including the file extension used to select the conversion route (params['ext'])
- Edited spectrum state triggering image regeneration

## Outputs

- spectrum.png — an automatically generated image of the spectra rendered from the uploaded data
- Low-resolution preview thumbnail for ELN display
- Higher-resolution PNG version suitable for reuse e.g. in publications
- Stored file set: original spectra, last edited version, and generated PNG

## How to apply

Send the uploaded spectrum file (JCAMP-DX .jdx/.dx/.jcamp or mzML-derived data) to the ChemSpectra transform endpoint /image; the controller wraps the upload in a FileContainer and passes it with form parameters (including params['ext']) to TransformerModel. TransformerModel.to_composer() selects the conversion route from the file extension, routing JCAMP-like input through jcamp2cv() to a JcampBaseConverter and then a TechniqueComposer (or MSComposer for MS data). The composer renders the spectrum via matplotlib-based logic at 200 dpi, and the controller returns the resulting spectrum.png via send_file. In an ELN embedding, produce two renderings — a low-resolution thumbnail for preview and a higher-resolution version suitable for reuse in publications — and regenerate both every time the user edits the spectrum so the stored PNG always reflects the last edited state.

## Related tools

- **ChemSpectra** (Backend service whose /image endpoint wraps the upload in a FileContainer, runs TransformerModel/TechniqueComposer, and returns the rendered spectrum.png via send_file) — https://github.com/ComPlat/chem-spectra-app
- **Chemotion ELN** (Main client that embeds ChemSpectra and stores the thumbnail and high-resolution PNG alongside the original and edited spectra) — https://github.com/ComPlat/chemotion_ELN
- **react-spectra-editor** (Front-end editor that displays the edited spectra whose last state the stored PNG must mirror) — https://github.com/ComPlat/react-spectra-editor

## Examples

```
curl -X POST -F 'file=@sample.jdx' -F 'ext=jdx' http://localhost:4000/api/spectra/image --output spectrum.png
```

## Evaluation signals

- spectrum.png exists in the file-management store and is a valid PNG rendered from the uploaded spectrum data
- The stored file set contains all three artifacts: the original spectra, the last edited version, and the generated PNG
- After a user edit, both images (thumbnail and higher-resolution version) are regenerated to match the edited spectrum
- The PNG is rendered by the matplotlib-based composer at 200 dpi
- Editing an MS spectrum routes through MSComposer while IR/NMR route through TechniqueComposer, both yielding a PNG

## Limitations

- Images must be regenerated on every edit; stale PNGs are a failure mode if the regeneration step is bypassed
- Only 1D 1H/13C NMR, IR and MS are covered — 2D data viewing/analysis is not supported
- Non-JCAMP-DX inputs require a first conversion step (e.g. RAW to mzML via msConvert, then mzML to JCAMP-DX via pymzML) before rendering
- No quantitative evaluation or performance metrics are reported for the rendering pipeline

## Evidence

- [abstract] File management stores original spectra, the last edited version, and an automatically generated png image: "The file management supports the storage of the original spectra along with the last edited version and an automatically generated image of the spectra in png format"
- [discussion] Storage of an automatically generated image of the spectra in png format: "and the storage of an automatically generated image of the spectra in png format"
- [readme] chem-spectra-app backend processes spectra from multiple formats: "This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files."
- [readme] Chemotion ELN as main client with react-spectra-editor: "Its main client is the [Chemotion ELN](https://github.com/ComPlat/chemotion_ELN), which displays and edits spectra with [react-spectra-editor](https://github.com/ComPlat/react-spectra-editor)."
