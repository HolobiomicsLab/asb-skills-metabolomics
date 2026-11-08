---
name: metabolomics-repository-quality-control
description: Use when you need to quantify validation errors across a corpus of mwTab
  files from the Metabolomics Workbench (e.g. schema violations, null section items,
  inconsistent SUBJECT_SAMPLE_FACTORS sample IDs) to guide repository curation.
when_to_use_negative:
- The input files are not in mwTab format (this skill's parser and JSON schemas are
  specific to the mwTab format used by Metabolomics Workbench for MS and NMR data).
- You need per-file interactive debugging of one dataset rather than corpus-wide error
  tallies for curation.
- The corpus has not been fetched via the MW REST interface or bulk download and you
  cannot enumerate the target studies.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3071
tools:
- name: mwtab
  role: Library used to read mwTab files (mwtab.fileio.read_files), represent them
    as MWTabFile objects, and run the validator (MWTabFile.validate / validate_file)
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
- name: jsonschema
  role: Backend for the validator's schema-based validation of mwTab files against
    ms_required_schema / nmr_required_schema
- name: pandas
  role: Aggregation of per-file validation results into category counts and export
    of the summary tallies
- name: Python
  role: Runtime for the validation pipeline (mwtab runs under Python 3.6+)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:773c9b9a703b2803f319b01ed765ca6fd69d1adcb72afd798387c72c3f5fd4aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/mwtab
  tools:
  - Python
  - jsonschema
  - mwtab
  - pandas
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# metabolomics-repository-quality-control

## Summary

Systematically validates every parseable mwTab-format dataset file in the Metabolomics Workbench repository using the mwtab validator (schema checks, null-value detection, sample ID consistency), then aggregates per-file error categories into tallies for curation and FAIRness assessment.

## When to use

Use when you need to quantify validation errors across a corpus of mwTab files from the Metabolomics Workbench (e.g. schema violations, null section items, inconsistent SUBJECT_SAMPLE_FACTORS sample IDs) to guide repository curation.

## When NOT to use

- The input files are not in mwTab format (this skill's parser and JSON schemas are specific to the mwTab format used by Metabolomics Workbench for MS and NMR data).
- You need per-file interactive debugging of one dataset rather than corpus-wide error tallies for curation.
- The corpus has not been fetched via the MW REST interface or bulk download and you cannot enumerate the target studies.

## Inputs

- mwTab-format dataset files for all Metabolomics Workbench studies (fetched via the MW REST interface or bulk download)
- mwschema.ms_required_schema and mwschema.nmr_required_schema JSON schema definitions bundled with the mwtab package

## Outputs

- Per-file validation results (schema errors, null section item values, sample ID inconsistencies, missing experimental data)
- Aggregated error-category tallies, e.g. counts written to validation_summary.csv (in the reference run: 1539 of 1818 parseable files had errors; 1272 with schema errors; 706 with null values)

## How to apply

Retrieve all mwTab-format dataset files for the Metabolomics Workbench studies via the MW REST interface or bulk download. Load each file with mwtab.fileio.read_files, recording and skipping any that fail to parse so the unparseable remainder of the corpus is tracked separately. For each parsed MWTabFile, call MWTabFile.validate with the mwschema.ms_required_schema and mwschema.nmr_required_schema defaults (jsonschema-backed) to detect schema violations, null values in section items, inconsistent SUBJECT_SAMPLE_FACTORS sample IDs, and absence of experimental data. Aggregate per-file validation results into category counts with pandas and write the error tallies (e.g. to validation_summary.csv) so error types that dominate the repository can be identified.

## Related tools

- **mwtab** (Library used to read mwTab files (mwtab.fileio.read_files), represent them as MWTabFile objects, and run the validator (MWTabFile.validate / validate_file)) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **jsonschema** (Backend for the validator's schema-based validation of mwTab files against ms_required_schema / nmr_required_schema)
- **pandas** (Aggregation of per-file validation results into category counts and export of the summary tallies)
- **Python** (Runtime for the validation pipeline (mwtab runs under Python 3.6+))

## Examples

```
import mwtab; files = [f for f in mwtab.read_files(*study_ids) if f]; errors = [f.validate(ms_schema=mwschema.ms_required_schema, nmr_schema=mwschema.nmr_required_schema) for f in files]
```

## Evaluation signals

- The count of skipped unparseable files plus parseable files equals the total corpus size (reference: 1818 parseable files).
- Each parseable file yields a validation verdict for all four checks: schema conformance, null section item values, SUBJECT_SAMPLE_FACTORS sample ID consistency, and presence of experimental data.
- Aggregated tallies are internally consistent: files with schema errors (1272 in the reference run) and files with null values (706) sum against the 1539 files with at least one validation error.
- Output table (e.g. validation_summary.csv) contains one row per error category with reproducible counts when the run is repeated on the same corpus.
- Files validated against mwschema.ms_required_schema vs mwschema.nmr_required_schema are matched to their experimental technique (MS vs NMR).

## Limitations

- Files that fail to parse with mwtab.fileio.read_files must be skipped and tracked separately, so validation coverage excludes the unparseable portion of the repository.
- Validation is schema-based and limited to the checks the validator implements (schema conformance, null values, sample ID consistency, absence of experimental data); other quality issues are not flagged.
- Results depend on the validator version (reference run used mwtab 1.0.1) and the bundled required_schema definitions.

## Evidence

- [abstract] Enhanced format validation evaluated over the full repository: "We used the enhanced format validation features to evaluate all available datasets in MW to facilitate improved curation and FAIRness of the repository."
- [methods] Validator API with ms/nmr required schemas: ".. automethod:: validate(ms_schema = mwschema.ms_required_schema, nmr_schema = mwschema.nmr_required_schema, verbose = True)"
- [methods] jsonschema backs the validation functionality: "jsonschema_ for validating functionality of ``mwTab`` files based on ``JSON`` schema."
- [methods] read_files is the corpus-loading entry point: ".. automodule:: mwtab.fileio

.. autofunction:: read_files"
- [readme] README quickstart shows read_files usage over study/analysis IDs: [evidence span withheld — anchor status: external_doc]
