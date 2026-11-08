---
name: mzml-to-jcamp-conversion
description: Reference for converting MS spectra stored in mzML or vendor RAW format
  into JCAMP-DX before visualization, peak detection, or ELN transfer with ChemSpectra/Chemotion.
when_to_use_negative:
- Input is already a JCAMP-DX file — it is read directly by the jcampconverter library
  in the frontend without this conversion step
- The target application is not ChemSpectra/Chemotion and already natively handles
  mzML or RAW — the conversion exists to satisfy ChemSpectra's standardized pipeline,
  not a general-purpose need
- Data are 1D 1H/13C NMR FID files — these go through an NMR-specific path, not the
  MS conversion chain
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0218
tools:
- name: pymzML
  role: Python parser used to read mzML files and build the JCAMP-DX output
- name: msconvert (ProteoWizard)
  role: Docker-container tool invoked by chem-spectra-app to convert proprietary RAW
    files to mzML before the pymzML-to-JCAMP-DX step
- name: chem-spectra-app
  role: Backend service that orchestrates the RAW→mzML→JCAMP-DX chain and exposes
    the converted spectrum to the frontend
  repo: https://github.com/ComPlat/chem-spectra-app
- name: react-spectra-editor
  role: Frontend that extracts and displays the resulting JCAMP-DX spectrum for visualization
    and editing
  repo: https://github.com/ComPlat/react-spectra-editor
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13321-020-00481-0
    title: 'ChemSpectra: a web-based spectra editor for analytical data'
  source_content_hash: sha256:c1efbec354047920f22f75836ff123f5b73a03c92116d63fea0cd2b98b7a9d3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ComPlat/chem-spectra-app
  tools:
  - ChemSpectra
  - Docker
  - NMRglue
  - Python
  - SciPy
  - chem-spectra-app
  - msconvert (ProteoWizard)
  - pymzML
  - react-spectra-editor
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# mzml-to-jcamp-conversion

## Summary

Converts mass-spectrometry data files (mzML, or proprietary Bruker/Thermo RAW via a ProteoWizard msconvert sidecar) into JCAMP-DX format so that spectra can be processed and standardized by the ChemSpectra editor.

## When to use

Reference for converting MS spectra stored in mzML or vendor RAW format into JCAMP-DX before visualization, peak detection, or ELN transfer with ChemSpectra/Chemotion.

## When NOT to use

- Input is already a JCAMP-DX file — it is read directly by the jcampconverter library in the frontend without this conversion step
- The target application is not ChemSpectra/Chemotion and already natively handles mzML or RAW — the conversion exists to satisfy ChemSpectra's standardized pipeline, not a general-purpose need
- Data are 1D 1H/13C NMR FID files — these go through an NMR-specific path, not the MS conversion chain

## Inputs

- mzML mass-spectrometry file
- proprietary vendor RAW file (e.g. Bruker/Thermo) convertible by ProteoWizard msconvert

## Outputs

- JCAMP-DX representation of the mass spectrum
- intermediate mzML file produced from RAW by msconvert
- MS spectrum ready for peak detection and display in react-spectra-editor

## How to apply

For mzML inputs, parse the file with pymzML (an open-source Python mass-spectrometry parser) and emit a composed JCAMP-DX representation via the MSConverter/MSComposer path in chem-spectra-app. For proprietary RAW files, first invoke the msconvert (ProteoWizard) tool running inside a Docker container, which the backend calls to obtain an mzML intermediate; then apply the same pymzML-to-JCAMP-DX conversion. After conversion, the standardized JCAMP-DX file is what gets classified (e.g., as MS) by the frontend extension mapping and is displayed/edited by react-spectra-editor. The rationale is that non-JCAMP-DX files are always processed into JCAMP-DX, allowing a standardized treatment of all data files after this first processing step.

## Related tools

- **pymzML** (Python parser used to read mzML files and build the JCAMP-DX output)
- **msconvert (ProteoWizard)** (Docker-container tool invoked by chem-spectra-app to convert proprietary RAW files to mzML before the pymzML-to-JCAMP-DX step)
- **chem-spectra-app** (Backend service that orchestrates the RAW→mzML→JCAMP-DX chain and exposes the converted spectrum to the frontend) — https://github.com/ComPlat/chem-spectra-app
- **react-spectra-editor** (Frontend that extracts and displays the resulting JCAMP-DX spectrum for visualization and editing) — https://github.com/ComPlat/react-spectra-editor

## Examples

```
msconvert --mzML -o out/ sample.raw && python -c "import pymzml; r=pymzml.run.Reader('out/sample.mzML'); print(r[0])"
```

## Evaluation signals

- The produced JCAMP-DX file contains at least one recognizable ##DATA TYPE= block that the frontend maps to the MS spectrum type
- The original RAW input was successfully converted to an mzML intermediate before JCAMP-DX generation (verified by the presence of that intermediate)
- After conversion, automatic peak detection with the default threshold runs without parsing errors
- The converted spectrum can be uploaded, read, and edited in react-spectra-editor without the editor falling back to an 'unknown' type
- The file is stored alongside its original spectra and a generated png image, per the ChemSpectra file-management rules

## Limitations

- Only MS data (mzML/RAW) are handled by this chain; NMR FID files follow a separate path and are not converted by the MS converters
- The RAW-to-mzML conversion relies on an external msconvert Docker sidecar, adding an operational dependency and failure point
- Future work notes that ChemSpectra 'should be extended to support further MS file formats', implying incomplete format coverage today

## Evidence

- [results] mzML-to-JCAMP-DX conversion via pymzML: "mzML files are converted to JCAMP-DX using pymzML, an Open Source python mass spectrometry file parser."
- [results] RAW files converted by msconvert in docker: "MsConvert in a docker container is called by the chem-spectra-app to achieve this job."
- [discussion] Standardization rationale: "All data files that are provided as non-JCAMP-DX files are processed and converted to JCAMP-DX, allowing a standardized treatment of all data files after a first processing step."
- [readme] Formats supported by backend: "This backend web service processes spectra -- NMR, IR, UV/VIS, MS, LC/MS, XRD, CV and more -- from JCAMP-DX, Bruker, BagIt, RAW and mzML files."
- [readme] Purpose of backend service: "chem-spectra-client is a standalone page that calls this service directly."
