---
name: lc-ms-data-ingestion-preprocessing
description: Use when starting a Nextflow4MS-DIAL (MS-DIAL/MS-FLO) run with vendor-proprietary
  LC-HRMS RAW files that must first be converted to .mzML or .abf before the pipeline's
  MS-DIAL processing step.
when_to_use_negative:
- Input files are already .mzML or .abf — just place them directly in data/raw_data/;
  no conversion is needed
- Data are already a peak/feature table — this skill only covers raw spectra ingestion
  before the MS-DIAL processing step
- The workflow is being used with formats other than mzML/ABF and you expect in-workflow
  conversion — the paper explicitly lists data file format conversion as a tool not
  yet incorporated
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: ProteoWizard msConvert
  role: Converts proprietary vendor RAW LC-MS files to the open .mzML format accepted
    by Nextflow4MS-DIAL
  repo: https://proteowizard.sourceforge.io/download.html
- name: Reifycs Abf Converter
  role: Alternative converter producing .abf files, the second raw-input extension
    the workflow accepts
  repo: https://www.reifycs.com/AbfConverter/
- name: Nextflow4MS-DIAL
  role: Downstream Nextflow workflow that ingests the converted mzML/ABF files from
    data/raw_data/ for MS-DIAL/MS-FLO processing
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:1c48353e6256195a6add2a4c55c724f6d8d3fb619c62903e94ce3c9bdca1bcce
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - Nextflow4MS-DIAL
  - ProteoWizard msConvert
  - ProteoWizard-msConvert
  - Reifycs Abf Converter
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# lc-ms-data-ingestion-preprocessing

## Summary

User-mediated conversion of proprietary vendor RAW LC-MS metabolomics files into the open mzML (ProteoWizard msConvert) or ABF (Reifycs Abf Converter) formats that Nextflow4MS-DIAL accepts in data/raw_data/, including filename hygiene and placement of parameter/library files.

## When to use

Use when starting a Nextflow4MS-DIAL (MS-DIAL/MS-FLO) run with vendor-proprietary LC-HRMS RAW files that must first be converted to .mzML or .abf before the pipeline's MS-DIAL processing step.

## When NOT to use

- Input files are already .mzML or .abf — just place them directly in data/raw_data/; no conversion is needed
- Data are already a peak/feature table — this skill only covers raw spectra ingestion before the MS-DIAL processing step
- The workflow is being used with formats other than mzML/ABF and you expect in-workflow conversion — the paper explicitly lists data file format conversion as a tool not yet incorporated

## Inputs

- Proprietary vendor RAW LC-MS data files (e.g., from the MetaboLights MTBLS733 raw deposit)
- ProteoWizard msConvert (for mzML conversion)
- Reifycs Abf Converter (for ABF conversion)
- MS-DIAL parameter file (msdial_params.txt)
- MS-FLO parameter file (msflo_params.ini)
- MS1 library (ms1_lib.txt)
- MS2 library (ms2_lib.msp)

## Outputs

- Converted .mzML files (or .abf files) in data/raw_data/ with special-character-free filenames
- Staged parameter and library files in data/ ready for the Nextflow4MS-DIAL pipeline

## How to apply

Obtain the vendor RAW files (e.g., the raw deposit of MetaboLights MTBLS733 by Li et al.) and rename them so they contain no special characters — underscores are safe, per the workflow FAQ — because unsafe filenames can silently prevent processes from executing. Run ProteoWizard msConvert to convert each RAW file to open mzML format, or use Reifycs Abf Converter to produce ABF files instead, since the workflow accepts both extensions. Place the converted spectra into data/raw_data/, the directory the workflow designates for raw input. Then stage the MS-DIAL and MS-FLO configuration files (msdial_params.txt, msflo_params.ini) and MS1/MS2 libraries (ms1_lib.txt, ms2_lib.msp) in data/ before launching the pipeline. Note this conversion step is deliberately outside the Nextflow workflow itself — the paper lists format conversion as future work — so it must be performed by the user before running `nextflow run`.

## Related tools

- **ProteoWizard msConvert** (Converts proprietary vendor RAW LC-MS files to the open .mzML format accepted by Nextflow4MS-DIAL) — https://proteowizard.sourceforge.io/download.html
- **Reifycs Abf Converter** (Alternative converter producing .abf files, the second raw-input extension the workflow accepts) — https://www.reifycs.com/AbfConverter/
- **Nextflow4MS-DIAL** (Downstream Nextflow workflow that ingests the converted mzML/ABF files from data/raw_data/ for MS-DIAL/MS-FLO processing) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial

## Examples

```
msconvert sample_01.RAW -o data/raw_data/ --mzML && nextflow run main.nf -profile docker > logs/execution.log
```

## Evaluation signals

- data/raw_data/ contains only .mzML (and/or .abf) files with no special characters in filenames (underscores only), satisfying the workflow FAQ
- The msConvert or Abf Converter run completes without error and output file count matches the number of input RAW files
- The workflow's input stage recognizes every converted file before the MS-DIAL processing step begins (no skipped processes due to naming)
- Configuration and library files are present and correctly named in data/: msdial_params.txt, msflo_params.ini, ms1_lib.txt, ms2_lib.msp
- A functional test (`nextflow run main.nf -profile functional_test`) succeeds with the staged inputs before processing real data

## Limitations

- Conversion is user-mediated, not automated inside the workflow; the paper lists format conversion tools as future work
- Filenames with special characters can cause processes not to execute after pipeline launch; only underscores are safe
- The workflow is command-line only and was tested on Intel-based macOS and Red Hat Enterprise Linux 8.8, not Apple silicon (M-series)
- MS-FLO must be downloaded separately and workflow component updates (e.g., MS-DIAL) are monitored manually

## Evidence

- [other] users convert them to mzML format with ProteoWizard-msConvert or to ABF format with Reifycs Abf Converter, after which the converted files serve as Nextflow4MS-DIAL inputs: [evidence span withheld — anchor status: not_found]
