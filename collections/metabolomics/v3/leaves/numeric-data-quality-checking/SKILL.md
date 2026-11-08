---
name: numeric-data-quality-checking
description: Use when quality-controlling processed MS or NMR metabolite data tables
  in mwTab format (e.g., MS_METABOLITE_DATA blocks) where zero-filled entries and
  degenerate rows/columns must be detected before deposition or curation.
when_to_use_negative:
- Input is raw instrument data or non-mwTab formats (e.g., mzML) that have not been
  parsed into MWTabFile objects
- Only schema/structure conformance is needed — use the jsonschema-driven MWTabFile.validate()
  path with ms_required_schema/nmr_required_schema instead
- Cross-checking sample IDs and factor headings between SUBJECT_SAMPLE_FACTORS and
  _DATA blocks — that is handled by the separate first sub-validation function
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_3174
tools:
- name: mwtab
  role: Provides the tokenizer, read_files generator, MWTabFile objects, and validate_file()
    numeric-value sub-validation that performs zero-to-null replacement and degenerate
    row/column detection
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
- name: pandas
  role: Used by mwtab for working with the tabular data sections within the mwTab
    format that the numeric checks operate on
- name: Python
  role: Runtime for the mwtab package (runs under Python 3.6+)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:2ad138014eaab200183743e564f4536f8a2471addba6ada1d55bc44aadf150cf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/mwtab
  tools:
  - Python
  - mwtab
  - pandas
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# numeric-data-quality-checking

## Summary

A numeric quality-control pass over processed metabolomics data tables parsed from mwTab files, as implemented in the mwtab package's validate_file() sub-validation functions: zeros are replaced with nulls and rows/columns dominated by a single non-null/non-zero value are flagged for curation.

## When to use

Use when quality-controlling processed MS or NMR metabolite data tables in mwTab format (e.g., MS_METABOLITE_DATA blocks) where zero-filled entries and degenerate rows/columns must be detected before deposition or curation.

## When NOT to use

- Input is raw instrument data or non-mwTab formats (e.g., mzML) that have not been parsed into MWTabFile objects
- Only schema/structure conformance is needed — use the jsonschema-driven MWTabFile.validate() path with ms_required_schema/nmr_required_schema instead
- Cross-checking sample IDs and factor headings between SUBJECT_SAMPLE_FACTORS and _DATA blocks — that is handled by the separate first sub-validation function

## Inputs

- Parsed MWTabFile objects containing mwTab-format study/analysis files (downloaded from the Metabolomics Workbench)
- Processed MS data tables within MS_METABOLITE_DATA (and NMR_METABOLITE_DATA) blocks

## Outputs

- Warnings/errors for rows and columns dominated by a single non-null/non-zero value
- Zero-to-null substituted data tables ready for schema validation
- Validation report (e.g., mwtab_validation_report.txt) when run with verbose=True

## How to apply

Parse mwTab-format study files downloaded from the Metabolomics Workbench into MWTabFile objects using the mwtab tokenizer and the read_files generator, then extract the tabular data sections (handled with pandas, which the package uses for 'working with tabular data sections within the mwTab format'). Run the numeric-value sub-validation of validate_file(): first replace zeros in the MS processed data tables with nulls, since zero entries in processed data are typically missing values rather than true measurements. Then scan the resulting matrix to detect rows or columns dominated by a single non-null/non-zero value, which indicates degenerate or unreliable measurements. Run this alongside the other three validate_file() sub-validators (sample ID/factor consistency, metabolite-name consistency, and jsonschema-based mwTab schema validation with the ms_required_schema and nmr_required_schema defaults) with verbose=True so all warnings and errors are captured, e.g. written to a validation report such as mwtab_validation_report.txt. The rationale is the paper's finding that a subset of sample ids from the SUBJECT_SAMPLE_FACTORS block must be present in the _DATA blocks, and that enhanced format validation was used to evaluate all datasets in the Metabolomics Workbench repository for curation and FAIRness.

## Related tools

- **mwtab** (Provides the tokenizer, read_files generator, MWTabFile objects, and validate_file() numeric-value sub-validation that performs zero-to-null replacement and degenerate row/column detection) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **pandas** (Used by mwtab for working with the tabular data sections within the mwTab format that the numeric checks operate on)
- **Python** (Runtime for the mwtab package (runs under Python 3.6+))

## Examples

```
from mwtab.validator import validate_file; validate_file(mwtabfile, ms_schema, nmr_schema, verbose=True)
```

## Evaluation signals

- No zero entries remain in the processed MS data tables after the zero-to-null substitution step
- All rows/columns dominated by a single non-null/non-zero value are reported when verbose=True is used
- Running validate_file(mwtabfile, ms_schema, nmr_schema, verbose=False) completes without unhandled exceptions on valid Metabolomics Workbench files
- A mwTab validation report containing warnings/errors is produced when all four sub-validators are run with verbose=True
- Sample IDs checked by the sibling sub-validator remain consistent: a subset of sample ids from SUBJECT_SAMPLE_FACTORS is present in the _DATA blocks

## Limitations

- The numeric checks apply to processed data tables in mwTab format used by the Metabolomics Workbench for MS and NMR archival; they are not a general-purpose QC tool for arbitrary tabular files
- Zero-to-null substitution assumes zeros in processed data represent missing rather than genuine zero measurements, which may misclassify true zero abundance values
- Coverage of what counts as 'dominated by a single non-null/non-zero value' is governed by the mwtab implementation and may flag legitimately sparse data for manual curation

## Evidence

- [methods] pandas for working with tabular data sections within the mwTab format.: "pandas_ for working with tabular data sections within the mwTab format."
- [methods] Validate data stored in mwTab file based on schema definition.: "Validate data stored in ``mwTab`` file based on schema definition."
- [readme] The mwtab package is a Python library that facilitates reading and writing files in mwTab format used by the Metabolomics Workbench.: "The ``mwtab`` package is a Python library that facilitates reading and writing
files in ``mwTab`` format used by the `Metabolomics Workbench`_"
