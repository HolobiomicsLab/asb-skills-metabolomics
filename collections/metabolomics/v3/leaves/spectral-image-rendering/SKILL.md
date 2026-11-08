---
name: spectral-image-rendering
description: Use when an uploaded spectrum (JCAMP-DX .jdx/.dx/.jcamp, Bruker zip,
  or mzML) must be accompanied by an automatically generated PNG image of the spectra
  as part of a file-management set in an ELN or repository workflow.
when_to_use_negative:
- The user needs interactive spectral analysis (peak-picking, integration, coupling
  constants) rather than a static image — use the react-spectra-editor front end instead
- The input is a proprietary RAW (MS) or FID (NMR) file that must first be converted
  (msConvert in docker, pymzML) before JCAMP-DX-standardized treatment
- The data is 2D NMR, which ChemSpectra does not yet support viewing or analyzing
edam_operation: http://edamontology.org/operation_0337
tools:
- name: ChemSpectra
  role: Backend web service whose /image endpoint and TransformerModel/JcampBaseConverter/TechniqueComposer
    pipeline render the spectrum PNG via matplotlib at 200 dpi
  repo: https://github.com/ComPlat/chem-spectra-app
- name: Chemotion ELN
  role: Host application whose file management stores the generated PNG alongside
    the original (*.zip) and last-edited (*.edit.jdx) spectra and triggers regeneration
    on each edit
  repo: https://github.com/ComPlat/chemotion_ELN
- name: react-spectra-editor
  role: JavaScript front end whose edits trigger regeneration of the thumbnail and
    high-resolution images; uses jcampconverter to parse the composed JCAMP-DX sent
    from the server side
  repo: https://github.com/ComPlat/react-spectra-editor
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:d53a058c96944b359857f37cc95dca2cd97f2c35341578c87872a981ad126308
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ComPlat/chem-spectra-app
  tools:
  - ChemSpectra
  - Chemotion ELN
  - JavaScript
  - react-spectra-editor
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# spectral-image-rendering

## Summary

Renders an automatically generated PNG image of a spectrum server-side using ChemSpectra's matplotlib-based composer layer, producing both a low-resolution thumbnail and a higher-resolution image (200 dpi) suitable for preview and publication reuse.

## When to use

Use when an uploaded spectrum (JCAMP-DX .jdx/.dx/.jcamp, Bruker zip, or mzML) must be accompanied by an automatically generated PNG image of the spectra as part of a file-management set in an ELN or repository workflow.

## When NOT to use

- The user needs interactive spectral analysis (peak-picking, integration, coupling constants) rather than a static image — use the react-spectra-editor front end instead
- The input is a proprietary RAW (MS) or FID (NMR) file that must first be converted (msConvert in docker, pymzML) before JCAMP-DX-standardized treatment
- The data is 2D NMR, which ChemSpectra does not yet support viewing or analyzing

## Inputs

- Uploaded spectrum file: JCAMP-DX (.jdx/.dx/.jcamp), Bruker zip, mzML, or other supported non-JCAMP-DX format
- Form parameters including params['ext'] used by TransformerModel to select the conversion route

## Outputs

- spectrum.png — server-rendered PNG image of the spectra (returned via send_file)
- Low-resolution thumbnail image for preview (ELN-embedded mode)
- Higher-resolution (200 dpi) image for reuse, e.g. in publications (ELN-embedded mode)

## How to apply

Send the uploaded spectrum file to the ChemSpectra transform endpoint /image: the controller wraps the upload in a FileContainer and passes it with extracted form parameters to TransformerModel. TransformerModel.to_composer() selects the conversion route from the file extension and params['ext'], routing JCAMP-like input through jcamp2cvp() to a JcampBaseConverter and then a TechniqueComposer (MSComposer for MS data). The composer renders the spectrum image at 200 dpi via the matplotlib-based rendering logic in the lib/composer layer, and the controller returns the generated spectrum.png via send_file. When embedded in an ELN such as Chemotion, generate two images — a low-resolution thumbnail for preview and a higher-resolution version for reuse, e.g. in publications — and regenerate both every time a user edits the spectrum, storing the PNG alongside the original and last-edited spectra (the *.edit.png of the original/*.edit.jdx/*.edit.png triple).

## Related tools

- **ChemSpectra** (Backend web service whose /image endpoint and TransformerModel/JcampBaseConverter/TechniqueComposer pipeline render the spectrum PNG via matplotlib at 200 dpi) — https://github.com/ComPlat/chem-spectra-app
- **Chemotion ELN** (Host application whose file management stores the generated PNG alongside the original (*.zip) and last-edited (*.edit.jdx) spectra and triggers regeneration on each edit) — https://github.com/ComPlat/chemotion_ELN
- **react-spectra-editor** (JavaScript front end whose edits trigger regeneration of the thumbnail and high-resolution images; uses jcampconverter to parse the composed JCAMP-DX sent from the server side) — https://github.com/ComPlat/react-spectra-editor

## Examples

```
curl -X POST -F 'file=@sample.jdx' http://<chem-spectra-app-host>/image -o spectrum.png
```

## Evaluation signals

- spectrum.png exists in the output directory and is a valid, non-empty PNG rendered from the uploaded spectrum data
- The image is rendered at 200 dpi by the matplotlib-based composer in the lib/composer layer
- Both the thumbnail and the higher-resolution image are regenerated after every user edit of the spectrum
- The output directory contains exactly the three-file set: the untouched original, the edited JCAMP-DX version, and the generated *.edit.png image

## Limitations

- Regeneration happens on every edit, so the PNG reflects only the last-edited state of the spectrum
- Only 1D 1H and 13C NMR, IR, and MS are covered by ChemSpectra; viewing and analysis of 2D data is listed as future improvement
- Proprietary RAW (MS) and FID (NMR) files require an extension/conversion step (msConvert docker container, pymzML) before standardized JCAMP-DX treatment

## Evidence

- [abstract] abstract: "The file management supports the storage of the original spectra along with the last edited version and an automatically generated image of the spectra in png format"
- [discussion] discussion: "and the storage of an automatically generated image of the spectra in png format"
- [methods] methods: "The jcampconverter library [17] inside the react-spectra-editor extracts spectroscopic data of the composed JCAMP-DX file which is sent from the server-side"
- [readme] readme: "This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files."
