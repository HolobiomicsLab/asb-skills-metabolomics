---
name: jcamp-dx-data-type-block-classification
description: Use when you need to automatically determine the spectra type (NMR, IR,
  MS, UV/VIS, XRD, CV, LC/MS) of an uploaded JCAMP-DX file before routing it to the
  correct conversion or analysis path in the chem-spectra-app backend.
when_to_use_negative:
- Input is a proprietary Bruker, RAW, FID, mzML, or BagIt file rather than a JCAMP-DX
  text file — those follow different conversion routes in chem-spectra-app (e.g. mzML
  via pymzML, RAW via msConvert).
- 'The file has already been converted to JCAMP-DX with an explicit, known ##DATA
  TYPE= value supplied by the user, so automated classification would add no information.'
- You need 2D NMR data viewing and analysis; the article lists 2D support as a missing
  signal not yet implemented.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3314
tools:
- name: ChemSpectra (chem-spectra-app backend)
  role: 'Implements the JcampBaseConverter over nmrglue''s read_blocks and the data_type.json
    mapping to classify ##DATA TYPE= blocks.'
  repo: https://github.com/ComPlat/chem-spectra-app
- name: NMRglue
  role: Python library used by the JcampBaseConverter to read JCAMP-DX blocks for
    classification.
- name: react-spectra-editor
  role: Front-end that automatically extracts the spectra type from the provided files
    when they are displayed, complementing the backend classification.
  repo: https://github.com/ComPlat/react-spectra-editor
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:fb8e36211d30cfc130abff9ae948cbf5098b1369603bbbc330ff3397c4f5a372
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ComPlat/chem-spectra-app
  tools:
  - ChemSpectra
  - ChemSpectra (chem-spectra-app backend)
  - Docker
  - NMRglue
  - Python
  - SciPy
  - msconvert (ProteoWizard)
  - pymzML
  - react-spectra-editor
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# jcamp-dx-data-type-block-classification

## Summary

Classifies JCAMP-DX spectra files into NMR, IR, MS, or unknown by parsing the ##DATA TYPE= blocks via nmrglue's read_blocks and a data_type.json mapping, as implemented in ComPlat/chem-spectra-app.

## When to use

Use when you need to automatically determine the spectra type (NMR, IR, MS, UV/VIS, XRD, CV, LC/MS) of an uploaded JCAMP-DX file before routing it to the correct conversion or analysis path in the chem-spectra-app backend.

## When NOT to use

- Input is a proprietary Bruker, RAW, FID, mzML, or BagIt file rather than a JCAMP-DX text file — those follow different conversion routes in chem-spectra-app (e.g. mzML via pymzML, RAW via msConvert).
- The file has already been converted to JCAMP-DX with an explicit, known ##DATA TYPE= value supplied by the user, so automated classification would add no information.
- You need 2D NMR data viewing and analysis; the article lists 2D support as a missing signal not yet implemented.

## Inputs

- JCAMP-DX spectra files with extensions .jcamp, .jdx, or .dx (IR, MS, NMR, UV/VIS, XRD, CV, LC/MS)
- data_type.json mapping file from chem-spectra-app

## Outputs

- spectrum_types.csv mapping each filename to its classified spectra type (NMR, IR, MS, UV/VIS, XRD, CV, LC/MS, or unknown)

## How to apply

Collect the JCAMP-DX files (extensions .jcamp, .jdx, .dx) into a working directory and open each with nmrglue's read_blocks through the JcampBaseConverter chain. Scan the blocks for the first recognised ##DATA TYPE= header, then look up that data type string in the data_type.json mapping shipped with chem-spectra-app to resolve it to one of the supported spectra types (NMR, IR, MS, UV/VIS, XRD, CV, LC/MS, or unknown). Record each filename with its assigned type in a mapping table and write it to spectrum_types.csv. If no ##DATA TYPE= block is recognised, classify the file as unknown rather than guessing. Note that the react-spectra-editor front-end extracts the spectra type automatically from the files sent by the server, so this backend classification should agree with the editor's auto-detected type.

## Related tools

- **ChemSpectra (chem-spectra-app backend)** (Implements the JcampBaseConverter over nmrglue's read_blocks and the data_type.json mapping to classify ##DATA TYPE= blocks.) — https://github.com/ComPlat/chem-spectra-app
- **NMRglue** (Python library used by the JcampBaseConverter to read JCAMP-DX blocks for classification.)
- **react-spectra-editor** (Front-end that automatically extracts the spectra type from the provided files when they are displayed, complementing the backend classification.) — https://github.com/ComPlat/react-spectra-editor

## Examples

```
python -c "import nmrglue as ng; d = ng.read_blocks(open('sample.jdx')); print(d)"
```

## Evaluation signals

- spectrum_types.csv exists with one row per input file and each type value is one of NMR, IR, MS, UV/VIS, XRD, CV, LC/MS, or unknown.
- The first recognised ##DATA TYPE= block in each JCAMP-DX file resolves to a non-empty mapping entry in data_type.json; unmatched types fall back to 'unknown' rather than raising an error.
- Backend classification matches the spectra type that react-spectra-editor auto-detects when the same file is opened for viewing.
- Files with extensions .jcamp, .jdx, and .dx are all routed through the same classification path; no file is silently skipped.

## Limitations

- Only the first recognised ##DATA TYPE= block is used; a JCAMP-DX file containing multiple blocks of mixed types will be classified by the first block alone.
- JCAMP-DX data types not present in data_type.json are reported as unknown with no further resolution.
- The article lists 2D NMR viewing/analysis as future work, so classification covers the 1D and other spectra types supported by the current backend.

## Evidence

- [methods] JcampBaseConverter over nmrglue's read_blocks classifies the spectrum from the first recognised ##DATA TYPE= block via the data_type.json mapping: [evidence span withheld — anchor status: not_found]
- [results] chem-spectra-app maps JCAMP-DX extensions to spectra types: [evidence span withheld — anchor status: not_found]
- [full_text] react-spectra-editor extracts spectra type automatically: [evidence span withheld — anchor status: not_found]
- [abstract] JCAMP-DX is an open file format supported by ChemSpectra: "The software can deal with the open file formats JCAMP-DX (IR, MS, NMR) and mzML (MS)"
