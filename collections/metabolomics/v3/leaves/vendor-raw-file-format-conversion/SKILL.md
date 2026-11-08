---
name: vendor-raw-file-format-conversion
description: Use when vendor-proprietary LC-MS raw files (e.g., the MTBLS733 raw deposit
  from Li et al.) must be converted to .mzML or .abf before ingestion into Nextflow4MS-DIAL's
  data/raw_data/ directory.
when_to_use_negative:
- Input files are already .mzML or .abf — the workflow accepts these directly in data/raw_data/
  and no conversion step is needed.
- Expecting the Nextflow4MS-DIAL workflow itself to perform conversion — conversion
  is user-mediated, not an automated process inside the pipeline.
- Data are already peak tables or post-MS-DIAL outputs — this skill only covers vendor
  raw-file ingestion, not downstream MS-DIAL/MS-FLO processing.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3273
- http://edamontology.org/topic_0092
tools:
- name: ProteoWizard msConvert
  role: Converts proprietary vendor raw LC-MS files to the open .mzML format required
    by the workflow's input stage
  repo: https://proteowizard.sourceforge.io/download.html
- name: Reifycs Abf Converter
  role: Alternative converter producing .abf files, which the workflow also accepts
    in data/raw_data/
  repo: https://www.reifycs.com/AbfConverter/
- name: Nextflow4MS-DIAL
  role: Consumer of the converted .mzML/.abf files; its input stage defines the accepted
    formats and directory layout
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:7f4f7d5f7a3ab74b5ea449b57a42ff52674b7122e4b3d756d03181139588bd5c
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
  - http://edamontology.org/topic_3273
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# vendor-raw-file-format-conversion

## Summary

Converts proprietary vendor LC-MS raw files into the open formats accepted by the Nextflow4MS-DIAL workflow — mzML via ProteoWizard msConvert or ABF via Reifycs Abf Converter — so they can be placed in data/raw_data/ for MS-DIAL processing.

## When to use

Use when vendor-proprietary LC-MS raw files (e.g., the MTBLS733 raw deposit from Li et al.) must be converted to .mzML or .abf before ingestion into Nextflow4MS-DIAL's data/raw_data/ directory.

## When NOT to use

- Input files are already .mzML or .abf — the workflow accepts these directly in data/raw_data/ and no conversion step is needed.
- Expecting the Nextflow4MS-DIAL workflow itself to perform conversion — conversion is user-mediated, not an automated process inside the pipeline.
- Data are already peak tables or post-MS-DIAL outputs — this skill only covers vendor raw-file ingestion, not downstream MS-DIAL/MS-FLO processing.

## Inputs

- Proprietary vendor LC-MS raw data files (e.g., vendor .RAW from the MetaboLights MTBLS733 raw deposit)
- ProteoWizard msConvert installation (for mzML output)
- Reifycs Abf Converter installation (for ABF output)

## Outputs

- Converted .mzML spectra files placed in data/raw_data/
- Optionally, .abf spectra files placed in data/raw_data/

## How to apply

Obtain the vendor RAW files (e.g., the raw deposit of MetaboLights MTBLS733 used to validate the workflow) and rename them so they contain no special characters, using underscores only, per the workflow FAQ. Run ProteoWizard msConvert on each RAW file to emit mzML, or use Reifycs Abf Converter to produce ABF files, writing the converted spectra into data/raw_data/. The Nextflow4MS-DIAL input stage accepts both .mzML and .abf extensions in data/raw_data/, so either target format satisfies the input requirement. Note that this conversion is user-mediated: it happens outside the workflow, and the paper lists format-conversion tooling as future work rather than an automated pipeline step. Only after converted files are in place should msdial_params.txt, msflo_params.ini, and MS1/MS2 libraries be added and the pipeline launched.

## Related tools

- **ProteoWizard msConvert** (Converts proprietary vendor raw LC-MS files to the open .mzML format required by the workflow's input stage) — https://proteowizard.sourceforge.io/download.html
- **Reifycs Abf Converter** (Alternative converter producing .abf files, which the workflow also accepts in data/raw_data/) — https://www.reifycs.com/AbfConverter/
- **Nextflow4MS-DIAL** (Consumer of the converted .mzML/.abf files; its input stage defines the accepted formats and directory layout) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial

## Evaluation signals

- data/raw_data/ contains converted .mzML (and/or .abf) files and no leftover proprietary RAW files
- All converted filenames are free of special characters (underscores only), otherwise the workflow FAQ warns processes may not execute
- The msConvert/Abf Converter run completes without error and each original RAW file has a corresponding converted output
- The workflow's input stage recognizes the converted files, evidenced by the pipeline proceeding past raw-data ingestion into MS-DIAL peak detection

## Limitations

- Conversion is not automated within the workflow: the paper's future-work list includes adding 'additional tools for metabolomics, such as those for data file format conversion', so users must run msConvert/Abf Converter manually.
- Special characters in file names can cause pipeline processes to silently not execute; only underscores are documented as safe.

## Evidence

- [readme] Convert other raw data formats to mzML with ProteoWizard msConvert, or create .abf files with Reifycs Abf Converter.: [evidence span withheld — anchor status: not_found]
