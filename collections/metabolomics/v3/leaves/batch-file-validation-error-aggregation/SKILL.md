---
name: batch-file-validation-error-aggregation
description: Use when a corpus of mwTab-format files (e.g. all Metabolomics Workbench
  studies) must be parsed, validated against required MS/NMR schemas, and rolled up
  into counts of error categories per file.
when_to_use_negative:
- Input files are not in mwTab format (the mwtab package reads and writes the mwTab
  format used by Metabolomics Workbench for MS and NMR archival data), e.g. already-JSONized
  or other metabolomics formats being validated with other schemas.
- The task is single-file format conversion (mwTab to JSON or vice versa) rather than
  corpus-wide quality-control error tallies.
- Files fail to parse in mwtab.fileio.read_files — these are recorded and skipped,
  not validated, so they should not be included in the error tallies.
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_3318
- http://edamontology.org/topic_0091
tools:
- name: mwtab
  role: Library providing fileio.read_files for batch loading mwTab files and MWTabFile.validate
    / validate_file for schema and null-value checks
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
- name: jsonschema
  role: Backing library for validating mwTab file data against the JSON schema definitions
    (ms_required_schema, nmr_required_schema)
- name: pandas
  role: Aggregation of per-file validation results into category counts written to
    validation_summary.csv
- name: Python
  role: Runtime environment (Python 3.6+) in which the mwtab library is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:918a6b58ab1e92abfce01605576b38b3d985f7a7d32b00330f96086f399a1d2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/mwtab
  tools:
  - Python
  - jsonschema
  - mwtab
  - pandas
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_3318
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# batch-file-validation-error-aggregation

## Summary

Batch-validate an entire repository corpus of mwTab-format files with the mwtab 1.0.1 validator (jsonschema-backed mwschema.ms_required_schema/nmr_required_schema) and aggregate per-file schema, null-value, and sample-ID errors into category tallies such as validation_summary.csv.

## When to use

Use when a corpus of mwTab-format files (e.g. all Metabolomics Workbench studies) must be parsed, validated against required MS/NMR schemas, and rolled up into counts of error categories per file.

## When NOT to use

- Input files are not in mwTab format (the mwtab package reads and writes the mwTab format used by Metabolomics Workbench for MS and NMR archival data), e.g. already-JSONized or other metabolomics formats being validated with other schemas.
- The task is single-file format conversion (mwTab to JSON or vice versa) rather than corpus-wide quality-control error tallies.
- Files fail to parse in mwtab.fileio.read_files — these are recorded and skipped, not validated, so they should not be included in the error tallies.

## Inputs

- mwTab-format dataset files from Metabolomics Workbench studies (fetched via the MW REST interface or bulk download)
- jsonschema definitions: mwschema.ms_required_schema and mwschema.nmr_required_schema

## Outputs

- Per-file validation results (schema violations, null section-item values, inconsistent SUBJECT_SAMPLE_FACTORS sample IDs, missing experimental data)
- Aggregated category counts of validation errors, e.g. written to validation_summary.csv

## How to apply

First retrieve the mwTab-format dataset files for all Metabolomics Workbench studies via the MW REST interface or bulk download. Load each file with mwtab.fileio.read_files, recording and skipping files that fail to parse so the parseable corpus size (e.g. 1818 files) is known. For each parsed MWTabFile call MWTabFile.validate using the default mwschema.ms_required_schema / nmr_required_schema schemas backed by the jsonschema library, checking for schema violations, null values in section items, inconsistent SUBJECT_SAMPLE_FACTORS sample IDs, and absence of experimental data. Finally aggregate the per-file validation results into category counts with pandas and write the summary tallies (e.g. 1539/1818 files with errors, 1272 with schema errors, 706 with null section-item values) to validation_summary.csv.

## Related tools

- **mwtab** (Library providing fileio.read_files for batch loading mwTab files and MWTabFile.validate / validate_file for schema and null-value checks) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **jsonschema** (Backing library for validating mwTab file data against the JSON schema definitions (ms_required_schema, nmr_required_schema))
- **pandas** (Aggregation of per-file validation results into category counts written to validation_summary.csv)
- **Python** (Runtime environment (Python 3.6+) in which the mwtab library is executed)

## Examples

```
for mwfile in mwtab.read_files(*study_ids): mwfile.validate(verbose=True)
```

## Evaluation signals

- The number of skipped unparseable files plus the number of validated files equals the total corpus size (e.g. 1818 parseable files in the reported run).
- Per-file validate() output flags each of the four error classes: schema violations, null values in section items, inconsistent SUBJECT_SAMPLE_FACTORS sample IDs, and absence of experimental data.
- Aggregated counts satisfy the invariant that total files-with-errors (1539) is the union of schema-error files (1272) and null-value files (706), so files-with-errors >= max(1272, 706) and <= 1272 + 706.
- validation_summary.csv contains the five reported tallies with no missing category rows.
- Re-running the pipeline yields identical counts, since validation is deterministic given the same corpus and schema versions.

## Limitations

- Files that fail to parse in mwtab.fileio.read_files cannot be validated and are skipped, so error tallies only cover the parseable subset of the corpus.
- Validation is limited to the schemas bundled with the mwtab version used (mwtab 1.0.1); results depend on the ms_required_schema/nmr_required_schema definitions available at run time.
- Tallies are corpus- and time-specific; counts change as new Metabolomics Workbench studies are deposited.

## Evidence

- [methods] validation against required MS/NMR schemas: ".. automethod:: validate(ms_schema = mwschema.ms_required_schema, nmr_schema = mwschema.nmr_required_schema, verbose = True)"
- [methods] batch file loading: ".. automodule:: mwtab.fileio

.. autofunction:: read_files"
- [methods] jsonschema-backed validation: "jsonschema_ for validating functionality of ``mwTab`` files based on ``JSON`` schema."
- [abstract] repository-wide quality-control evaluation: "We used the enhanced format validation features to evaluate all available datasets in MW to facilitate improved curation and FAIRness of the repository."
- [readme] README quickstart usage of read_files: ">>> for mwfile in mwtab.read_files("1", "2"):
...      print("STUDY_ID:", mwfile.study_id)"
