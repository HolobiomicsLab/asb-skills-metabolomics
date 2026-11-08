---
name: metabolomics-filename-sanitization-for-workflow-inputs
description: Use when preparing proprietary vendor RAW LC-MS files for Nextflow4MS-DIAL
  ingestion, before placing converted .mzML or .abf files into data/raw_data/ for
  processing.
when_to_use_negative:
- Input files are already .mzML or .abf with underscore-only filenames — they can
  be placed directly in data/raw_data/ without sanitization or conversion
- You are past the raw-data ingestion stage and working with workflow outputs (e.g.
  the CSV peak tables or .msdial/.tsv files in results/) — filename sanitization applies
  to raw vendor inputs, not downstream tables
- You expect the workflow itself to convert proprietary formats — Nextflow4MS-DIAL
  does not bundle a conversion tool (the paper lists data file format conversion as
  future work)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: ProteoWizard msConvert
  role: Converts sanitized vendor RAW LC-MS files to the .mzML format accepted by
    Nextflow4MS-DIAL's data/raw_data/ input stage
  repo: https://proteowizard.sourceforge.io/download.html
- name: Reifycs Abf Converter
  role: Alternative converter producing .abf files, the second raw-input extension
    the workflow accepts
  repo: https://www.reifycs.com/AbfConverter/
- name: Nextflow4MS-DIAL
  role: The downstream Nextflow workflow whose data/raw_data/ input stage and FAQ
    filename rules define the sanitization requirements
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:ad9e493fbb96f5ddff041a84674446e82ca0d75aa6ee37b6cf2aee91b906a4c5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Nextflow4MS-DIAL
  - ProteoWizard msConvert
  - ProteoWizard-msConvert
  - Reifycs Abf Converter
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metabolomics-filename-sanitization-for-workflow-inputs

## Summary

A user-mediated pre-processing step for the Nextflow4MS-DIAL workflow: vendor RAW LC-MS metabolomics files must be renamed with underscores replacing special characters, then converted to mzML (ProteoWizard msConvert) or ABF (Reifycs Abf Converter) before being placed in data/raw_data/.

## When to use

Use when preparing proprietary vendor RAW LC-MS files for Nextflow4MS-DIAL ingestion, before placing converted .mzML or .abf files into data/raw_data/ for processing.

## When NOT to use

- Input files are already .mzML or .abf with underscore-only filenames — they can be placed directly in data/raw_data/ without sanitization or conversion
- You are past the raw-data ingestion stage and working with workflow outputs (e.g. the CSV peak tables or .msdial/.tsv files in results/) — filename sanitization applies to raw vendor inputs, not downstream tables
- You expect the workflow itself to convert proprietary formats — Nextflow4MS-DIAL does not bundle a conversion tool (the paper lists data file format conversion as future work)

## Inputs

- Proprietary vendor RAW LC-MS metabolomics data files (e.g. from the MetaboLights MTBLS733 raw deposit)
- Converted .mzML or .abf files destined for data/raw_data/

## Outputs

- Special-character-free filenames (underscores only) for all raw input files
- .mzML files produced by ProteoWizard msConvert and/or .abf files produced by Reifycs Abf Converter, placed in data/raw_data/

## How to apply

Take each vendor RAW LC-MS file (e.g. from the raw deposit of the MetaboLights MTBLS733 study by Li et al. used to validate the workflow) and rename it so it contains no special characters, using underscores only, because special characters in filenames cause Nextflow4MS-DIAL processes to silently not execute. Then convert each sanitized RAW file to the open mzML format with ProteoWizard msConvert, or produce ABF-format files with the Reifycs Abf Converter, since the workflow's input stage accepts only .mzML and .abf extensions. Place the converted files in data/raw_data/ (removing the bundled example data first), add msdial_params.txt, msflo_params.ini, ms1_lib.txt and ms2_lib.msp to data/, and run the pipeline with a docker (local) or singularity (HPC) profile. The rationale is that this conversion and renaming is user-mediated, not automated inside the workflow: the paper explicitly lists data file format conversion as a future-work tool, so sanitization and conversion must happen before `nextflow run` is invoked.

## Related tools

- **ProteoWizard msConvert** (Converts sanitized vendor RAW LC-MS files to the .mzML format accepted by Nextflow4MS-DIAL's data/raw_data/ input stage) — https://proteowizard.sourceforge.io/download.html
- **Reifycs Abf Converter** (Alternative converter producing .abf files, the second raw-input extension the workflow accepts) — https://www.reifycs.com/AbfConverter/
- **Nextflow4MS-DIAL** (The downstream Nextflow workflow whose data/raw_data/ input stage and FAQ filename rules define the sanitization requirements) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial

## Examples

```
nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- Every input filename contains only alphanumeric characters and underscores, per the workflow FAQ rule
- data/raw_data/ contains converted files with .mzML and/or .abf extensions and nothing with the original vendor RAW extension
- The number of converted files in data/raw_data/ equals the number of source RAW files (no file lost in renaming/conversion)
- After `nextflow run`, no process is silently skipped — the FAQ symptom 'a process was not executed after pipeline execution' is absent
- logs/execution.log records a successful run with the expected MS-DIAL processing (e.g. peak detection over the ingested files)

## Limitations

- Special characters in file names cause processes not to execute after pipeline launch; only underscores are documented as safe
- Conversion is user-mediated: the workflow does not automate RAW-to-mzML/ABF conversion, and incorporating format-conversion tools is listed as future work
- File handling and renaming are documented for the Nextflow4MS-DIAL context (tested on macOS 13.5.1 Intel and Red Hat Enterprise Linux 8.8 / HiPerGator), not as a general-purpose renaming procedure

## Evidence

- [readme] Conversion tooling prescribed in the README: [evidence span withheld — anchor status: not_found]
