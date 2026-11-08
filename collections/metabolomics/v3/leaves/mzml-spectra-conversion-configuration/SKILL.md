---
name: mzml-spectra-conversion-configuration
description: Use when proprietary vendor LC-MS metabolomics raw files must be converted
  to open .mzML or .abf format and staged in data/raw_data/ before running Nextflow4MS-DIAL.
when_to_use_negative:
- Input files are already in .mzML or .abf format and simply need to be placed in
  data/raw_data/ — no conversion is needed
- A fully automated, in-workflow format-conversion step is required; the workflow
  authors list data file format conversion as future work, not a built-in step
- Filenames still contain special characters and have not been sanitized — renaming
  must precede conversion or downstream processes may not execute
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: ProteoWizard msConvert
  role: Converts proprietary vendor RAW LC-MS files to the open .mzML format accepted
    by the workflow
  repo: https://proteowizard.sourceforge.io/download.html
- name: Reifycs Abf Converter
  role: Alternative converter producing .abf files, the second accepted raw-input
    format
  repo: https://www.reifycs.com/AbfConverter/
- name: Nextflow4MS-DIAL
  role: Downstream workflow whose input stage consumes the converted .mzML/.abf files
    from data/raw_data/
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:6c0813633b33bf14c00e51a22dc60a1fb7370c84f0002cecf1aa13353be2b7d4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Nextflow4MS-DIAL
  - ProteoWizard msConvert
  - ProteoWizard-msConvert
  - Reifycs Abf Converter
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mzml-spectra-conversion-configuration

## Summary

User-mediated conversion of proprietary vendor LC-HRMS raw files into open formats (.mzML via ProteoWizard msConvert, or .abf via Reifycs Abf Converter) so they can be ingested by the Nextflow4MS-DIAL workflow.

## When to use

Use when proprietary vendor LC-MS metabolomics raw files must be converted to open .mzML or .abf format and staged in data/raw_data/ before running Nextflow4MS-DIAL.

## When NOT to use

- Input files are already in .mzML or .abf format and simply need to be placed in data/raw_data/ — no conversion is needed
- A fully automated, in-workflow format-conversion step is required; the workflow authors list data file format conversion as future work, not a built-in step
- Filenames still contain special characters and have not been sanitized — renaming must precede conversion or downstream processes may not execute

## Inputs

- Proprietary vendor RAW LC-MS data files (e.g., from the MetaboLights MTBLS733 raw deposit)
- ProteoWizard msConvert installation
- Reifycs Abf Converter installation (optional, for ABF output)

## Outputs

- .mzML spectra files in data/raw_data/ (or .abf files as an alternative)
- Special-character-free filenames using underscores

## How to apply

Nextflow4MS-DIAL does not automate format conversion; users convert files outside the workflow first. Run ProteoWizard msConvert on each vendor RAW file to produce .mzML, or use the Reifycs Abf Converter to produce .abf — the workflow's input stage accepts both extensions in data/raw_data/. Before conversion, rename files to remove special characters, using underscores only, since special characters in filenames can cause Nextflow processes to be skipped unexpectedly. Place the converted files in data/raw_data/ (replacing the example dataset), then add msdial_params.txt, msflo_params.ini, ms1_lib.txt, and ms2_lib.msp to data/ before launching with the docker or singularity profile. The rationale is that MS-DIAL processes .mzML LC-MS metabolomics data within the containerized workflow (MS-DIAL -> MS-FLO), so only these open formats reach the pipeline.

## Related tools

- **ProteoWizard msConvert** (Converts proprietary vendor RAW LC-MS files to the open .mzML format accepted by the workflow) — https://proteowizard.sourceforge.io/download.html
- **Reifycs Abf Converter** (Alternative converter producing .abf files, the second accepted raw-input format) — https://www.reifycs.com/AbfConverter/
- **Nextflow4MS-DIAL** (Downstream workflow whose input stage consumes the converted .mzML/.abf files from data/raw_data/) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial

## Evaluation signals

- data/raw_data/ contains converted files with .mzML (or .abf) extensions before the pipeline is launched
- All converted filenames contain no special characters (underscores only), so no Nextflow process is skipped after execution
- The workflow's input stage recognizes the converted files as raw LC-MS metabolomics data prior to MS-DIAL processing
- The conversion step runs without error and downstream MS-DIAL peak detection proceeds (e.g., MS-DIAL identified 25,447 peaks on the MTBLS733 test data)

## Limitations

- Conversion is user-mediated, not automated within the workflow; the article notes future work could incorporate 'tools for ... data file format conversion'
- Special characters in file names can cause Nextflow processes to silently not execute; underscores are the documented safe choice
- The workflow has only been tested on macOS 13.5.1 (Intel) and Red Hat Enterprise Linux 8.8, not Apple silicon

## Evidence
