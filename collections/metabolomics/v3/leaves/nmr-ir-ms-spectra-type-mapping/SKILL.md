---
name: nmr-ir-ms-spectra-type-mapping
description: Use when a batch of spectra files (JCAMP-DX, mzML, Bruker, RAW, FID,
  BagIt or ZIP) must be automatically classified as NMR, IR, MS or unknown before
  further conversion or rendering.
when_to_use_negative:
- The file already carries an explicit, trusted spectra-type annotation and only needs
  downstream conversion or peak-picking, not classification.
- The input is 2D NMR data or another spectra type outside the one-dimensional 1H/13C
  NMR, IR and MS scope that ChemSpectra covers; the article lists 2D support as future
  work.
- The goal is quantitative spectral analysis (integration, coupling constants, multiplicity)
  rather than determining which type a file represents.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0003
tools:
- name: ChemSpectra
  role: 'Backend (chem-spectra-app) that performs the extension- and ##DATA TYPE=-based
    classification of each spectra file'
  repo: github.com/ComPlat/chem-spectra-app
- name: NMRglue
  role: Python library whose read_blocks routine underlies JCAMP-DX block reading
    used for type detection
- name: pymzML
  role: Open-source Python mass spectrometry parser used in the MS route for mzML
    handling
- name: msconvert (ProteoWizard)
  role: Docker sidecar invoked by chem-spectra-app to convert proprietary RAW MS files
    to mzML before type mapping
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:111740add6484909f632ee763a03c59f90e5fa259d75e65cfc871d861bdbdadb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChemSpectra
  - Docker
  - NMRglue
  - Python
  - SciPy
  - msconvert (ProteoWizard)
  - pymzML
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0003
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# nmr-ir-ms-spectra-type-mapping

## Summary

Maps uploaded analytical spectra files to their spectra type — NMR, IR, MS, UV/VIS, XRD or CV — by combining file-extension conventions and ##DATA TYPE= header inspection as implemented in the ChemSpectra chem-spectra-app backend.

## When to use

Use when a batch of spectra files (JCAMP-DX, mzML, Bruker, RAW, FID, BagIt or ZIP) must be automatically classified as NMR, IR, MS or unknown before further conversion or rendering.

## When NOT to use

- The file already carries an explicit, trusted spectra-type annotation and only needs downstream conversion or peak-picking, not classification.
- The input is 2D NMR data or another spectra type outside the one-dimensional 1H/13C NMR, IR and MS scope that ChemSpectra covers; the article lists 2D support as future work.
- The goal is quantitative spectral analysis (integration, coupling constants, multiplicity) rather than determining which type a file represents.

## Inputs

- JCAMP-DX files with extensions .jcamp, .jdx or .dx (NMR/IR/MS)
- mzML files (MS)
- proprietary RAW MS files
- FID or ZIP files (NMR)
- Bruker and BagIt packages
- data_type.json extension-to-type mapping

## Outputs

- spectrum_types.csv mapping each filename to NMR, IR, MS or unknown
- typed JCAMP-DX representation of each spectrum
- status per file (converted successfully / unknown type / conversion failure)

## How to apply

For each JCAMP-DX file, run the JcampBaseConverter chain (which reads blocks via nmrglue's read_blocks) and classify the spectrum from the first recognised ##DATA TYPE= block, resolving the type through the backend's data_type.json mapping. Route mzML files through the MS conversion path (MSConverter/MSComposer built on pymzML) so they are treated as mass spectra; proprietary RAW MS inputs must first be converted to mzML by the msconvert (ProteoWizard) docker sidecar before classification. FID or ZIP inputs follow the NMR route. Record each filename together with its assigned type in a mapping table, writing values restricted to NMR, IR, MS or unknown. The rationale is that ChemSpectra standardises treatment by converting all non-JCAMP-DX data to JCAMP-DX as a first processing step, and classification precedes any editor-side rendering by react-spectra-editor.

## Related tools

- **ChemSpectra** (Backend (chem-spectra-app) that performs the extension- and ##DATA TYPE=-based classification of each spectra file) — github.com/ComPlat/chem-spectra-app
- **NMRglue** (Python library whose read_blocks routine underlies JCAMP-DX block reading used for type detection)
- **pymzML** (Open-source Python mass spectrometry parser used in the MS route for mzML handling)
- **msconvert (ProteoWizard)** (Docker sidecar invoked by chem-spectra-app to convert proprietary RAW MS files to mzML before type mapping)

## Examples

```
python scripts/map_spectrum_types.py --data_type_json data_type.json --inputs '*.jdx' '*.mzML' '*.raw' --out spectrum_types.csv
```

## Evaluation signals

- spectrum_types.csv exists with exactly one row per input file
- each type value is one of NMR, IR, MS or unknown (no free-text types)
- every .jcamp/.jdx/.dx file resolves through data_type.json without an 'unknown' unless the ##DATA TYPE= block is unrecognised
- RAW MS inputs appear in the table only after successful msconvert-to-mzML conversion; files whose msconvert fails should be marked unknown, not silently dropped

## Limitations

- Files whose ##DATA TYPE= block does not map to a known type in data_type.json fall to 'unknown'; the mapping is extension-driven and may misclassify non-standard extensions.
- Support for further MS file formats is explicitly future work; formats outside JCAMP-DX, Bruker, BagIt, RAW, FID and mzML are not handled.
- 2D NMR data and spectra-type comparison across windows are listed by the authors as future improvements, so 2D datasets cannot be reliably classified by this mapping.

## Evidence

- [other] react-spectra-editor automatically extracts the spectra type from the provided files: [evidence span withheld — anchor status: not_found]
- [other] chem-spectra-app maps file extensions jcamp, jdx, dx to NMR/IR/MS and mzML/RAW to MS, FID or ZIP to NMR: [evidence span withheld — anchor status: not_found]
- [results] mzML conversion to JCAMP-DX via pymzML: "mzML files are converted to JCAMP-DX using pymzML, an Open Source python mass spectrometry file parser."
- [results] MsConvert in docker called by chem-spectra-app for RAW conversion: "MsConvert in a docker container is called by the chem-spectra-app to achieve this job."
- [readme] README: backend processes spectra from JCAMP-DX, Bruker, BagIt, RAW and mzML: "This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files."
- [discussion] Non-JCAMP-DX files converted to JCAMP-DX as first processing step: "All data files that are provided as non-JCAMP-DX files are processed and converted to JCAMP-DX, allowing a standardized treatment of all data files after a first processing step."
