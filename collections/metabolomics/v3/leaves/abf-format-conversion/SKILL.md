---
name: abf-format-conversion
description: Use when LC-HRMS metabolomics data arrives in proprietary vendor formats
  and must be made ingestible by a pipeline that only accepts .mzML or .abf input
  files.
when_to_use_negative:
- Input files are already in .mzML or .abf format and placed in data/raw_data/ — the
  workflow's input stage already accepts these and no conversion is needed
- The analysis goal is automated, in-pipeline format conversion as part of Nextflow4MS-DIAL
  — the paper explicitly lists data file format conversion as a tool not yet incorporated
  into the workflow
- Working with non-LC-MS data (e.g., genomics or LC-MS/MS proteomics pipelines) where
  mzML/ABF metabolomics ingestion is not the target schema
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_3334
tools:
- name: Reifycs Abf Converter
  role: External converter used to create .abf files as an alternative to mzML, since
    the workflow accepts both .mzML and .abf extensions in data/raw_data/
- name: ProteoWizard-msConvert
  role: The complementary converter for producing the open mzML format that Nextflow4MS-DIAL
    primarily ingests; invoked outside the workflow by the user
- name: Nextflow4MS-DIAL
  role: The consuming workflow whose input stage requires converted .mzML or .abf
    files in data/raw_data/ before MS-DIAL processing
  repo: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/jasms.4c00364
    title: 'Nextflow4MS-DIAL: A Reproducible Nextflow-Based Workflow for Liquid Chromatography–Mass
      Spectrometry Metabolomics Data Processing'
  source_content_hash: sha256:8ebcc269d7f52617bfe39108fbbf3c28bbf060369e735015404201fcf319206d
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/Nextflow4Metabolomics/nextflow4ms-dial
  tools:
  - Nextflow4MS-DIAL
  - ProteoWizard-msConvert
  - Reifycs Abf Converter
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# abf-format-conversion

## Summary

A user-mediated raw-file ingestion step for the Nextflow4MS-DIAL LC-MS metabolomics workflow: proprietary vendor RAW files are converted outside the pipeline into the open mzML format with ProteoWizard msConvert, or into ABF format with Reifycs Abf Converter, before placement in data/raw_data/.

## When to use

Use when LC-HRMS metabolomics data arrives in proprietary vendor formats and must be made ingestible by a pipeline that only accepts .mzML or .abf input files.

## When NOT to use

- Input files are already in .mzML or .abf format and placed in data/raw_data/ — the workflow's input stage already accepts these and no conversion is needed
- The analysis goal is automated, in-pipeline format conversion as part of Nextflow4MS-DIAL — the paper explicitly lists data file format conversion as a tool not yet incorporated into the workflow
- Working with non-LC-MS data (e.g., genomics or LC-MS/MS proteomics pipelines) where mzML/ABF metabolomics ingestion is not the target schema

## Inputs

- Proprietary vendor RAW LC-MS metabolomics data files (e.g., Thermo/ABF-style raw deposits, such as those from MetaboLights MTBLS733)
- Reifycs Abf Converter software for producing ABF-format files

## Outputs

- .abf files (or .mzML files produced via the complementary msConvert path) placed in data/raw_data/ with special-character-free, underscore-only filenames

## How to apply

Obtain the proprietary vendor RAW LC-MS files (e.g., from the raw deposit of the MetaboLights MTBLS733 study by Li et al.) and rename them so they contain no special characters, using underscores only, per the workflow FAQ. Run ProteoWizard msConvert on each RAW file to write open-format mzML spectra into data/raw_data/, the directory Nextflow4MS-DIAL designates for raw input. Alternatively, use the Reifycs Abf Converter to produce .abf files instead, since the workflow accepts both .mzML and .abf extensions in data/raw_data/. The rationale is that conversion is deliberately not automated within the Nextflow workflow — the paper notes format-conversion tools are future work — so the practitioner performs this step externally before running `nextflow run main.nf`, ensuring the input stage recognizes the files prior to the MS-DIAL processing step.

## Related tools

- **Reifycs Abf Converter** (External converter used to create .abf files as an alternative to mzML, since the workflow accepts both .mzML and .abf extensions in data/raw_data/)
- **ProteoWizard-msConvert** (The complementary converter for producing the open mzML format that Nextflow4MS-DIAL primarily ingests; invoked outside the workflow by the user)
- **Nextflow4MS-DIAL** (The consuming workflow whose input stage requires converted .mzML or .abf files in data/raw_data/ before MS-DIAL processing) — https://github.com/Nextflow4Metabolomics/nextflow4ms-dial

## Evaluation signals

- data/raw_data/ contains converted .abf (and/or .mzML) files that the workflow's input stage recognizes
- All converted filenames contain no special characters — underscores only — satisfying the workflow FAQ requirement
- The conversion command runs without error and produces files with valid .abf extensions
- The subsequent `nextflow run main.nf` execution proceeds past the input stage into MS-DIAL processing (e.g., logs/execution.log shows the input processes executed rather than being skipped)

## Limitations

- Conversion is user-mediated, not automated within the Nextflow workflow; the paper lists data file format conversion among tools not yet incorporated
- Special characters in filenames cause processes to not execute — the FAQ warns against them and recommends underscores
- The workflow itself is command-line only, so no GUI exists to assist users with the conversion-to-ingestion sequence

## Evidence

- [readme] README conversion instructions: "Convert other raw data formats to `.mzML` with [ProteoWizard msConvert](https://proteowizard.sourceforge.io/download.html), or create `.abf` files with [Reifycs Abf Converter]"
- [readme] README accepted input formats: "Remove the example data and place your raw data files in `data/raw_data/`. The workflow accepts `.mzML` and `.abf` files."
- [readme] README FAQ on filenames: "To avoid unexpected errors, do not use special characters in file names. Underscores are safe to use."
- [discussion] Paper discussion on format conversion as future work: "future work on developments could incorporate additional tools for metabolomics, such as those for data file format conversion, statistical analysis, and pathway analysis"
