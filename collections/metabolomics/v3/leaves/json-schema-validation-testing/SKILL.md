---
name: json-schema-validation-testing
description: Use when auditing or reproducing the automated test coverage of a Python
  package that validates extracted JSON against JSON schemas (Experiment Description
  Specification, Protocol Dependent Schema, format-specific schemas), or when establishing
  a >90% coverage baseline for schema-validation code.
when_to_use_negative:
- The goal is functional validation of user data files (e.g., checking a tagged-extracted
  JSON against a project schema with the validate command) rather than measuring the
  test coverage of the validation code itself.
- The package under test is not a Python 3 package installable via pip with an existing
  pytest/unittest suite — coverage.py measurement against the >90% MESSES baseline
  does not transfer.
- The target output format is not among the supported conversions (currently mwTab
  for Metabolomics Workbench), so the format-specific schema testing path cannot be
  exercised.
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MESSES
  role: Package under test; provides the extract, validate, and convert commands and
    the modules whose coverage is measured (extract.extract, validate.validate, convert.convert,
    user_input_checking, mwTab_functions)
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: Python 3
  role: Runtime environment (3.10+) in which the package is installed and the test
    suite is executed under coverage.py
- name: jsonschema
  role: Dependency backing the validate command's schema checking, whose validation
    behavior is exercised by the test suite
- name: mwtab
  role: Dependency used by messes.convert.mwTab_functions for conversion to the mwTab
    format accepted by Metabolomics Workbench
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:321dc61066011cd8599c795a9ef3e0a00c70fe0966bfaae03f3a92a61e97d8ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/messes
  tools:
  - MESSES
  - Python 3
  - jsonschema
  - mwtab
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# json-schema-validation-testing

## Summary

Verify that the MESSES package's JSON-schema-based validation and conversion code is well tested by running its full unit-test suite under coverage.py and confirming total code coverage above 90% across all modules (extract, validate, convert, user_input_checking, mwTab conversion functions).

## When to use

Use when auditing or reproducing the automated test coverage of a Python package that validates extracted JSON against JSON schemas (Experiment Description Specification, Protocol Dependent Schema, format-specific schemas), or when establishing a >90% coverage baseline for schema-validation code.

## When NOT to use

- The goal is functional validation of user data files (e.g., checking a tagged-extracted JSON against a project schema with the validate command) rather than measuring the test coverage of the validation code itself.
- The package under test is not a Python 3 package installable via pip with an existing pytest/unittest suite — coverage.py measurement against the >90% MESSES baseline does not transfer.
- The target output format is not among the supported conversions (currently mwTab for Metabolomics Workbench), so the format-specific schema testing path cannot be exercised.

## Inputs

- Installed MESSES package source tree (Python 3.10+, from PyPI or MoseleyBioinformaticsLab/MESSES)
- The package's automated unit-test suite as configured in its GitHub Actions CI (build.yml workflow)
- coverage.py tool for coverage measurement
- Sample JSON data and schemas (Experiment Description Specification, Protocol Dependent Schema, format-specific schema) exercised by the validate command

## Outputs

- coverage_report.json — machine-readable per-module line/statement coverage and total coverage percentage
- Total code coverage percentage compared against the reported >90% threshold (met/not-met verdict)
- Per-module coverage values for messes.extract.extract, messes.validate.validate, messes.convert.convert, messes.convert.user_input_checking, and messes.convert.mwTab_functions

## How to apply

Install the MESSES package (Python 3.10+, via pip from PyPI or the MoseleyBioinformaticsLab/MESSES GitHub source) in a Linux environment. Execute the full automated unit-test suite — the same suite wired into the package's continuous integration under GitHub Actions — under the coverage.py tool so that every module is exercised: messes.extract.extract, messes.validate.validate, messes.convert.convert, messes.convert.user_input_checking, and messes.convert.mwTab_functions (the validate command tests JSON data against the Experiment Description Specification, any Protocol Dependent Schema the user supplies via --pds, additional user JSON schemas, and a format-specific built-in schema). Emit the machine-readable coverage report as coverage_report.json, recording line/statement coverage per module and the total coverage percentage. Compare the measured total percentage against the paper's reported figure of above 90% and record whether the threshold is met; the package also publishes a Codecov badge tracking branch coverage on main.

## Related tools

- **MESSES** (Package under test; provides the extract, validate, and convert commands and the modules whose coverage is measured (extract.extract, validate.validate, convert.convert, user_input_checking, mwTab_functions)) — https://github.com/MoseleyBioinformaticsLab/messes
- **Python 3** (Runtime environment (3.10+) in which the package is installed and the test suite is executed under coverage.py)
- **jsonschema** (Dependency backing the validate command's schema checking, whose validation behavior is exercised by the test suite)
- **mwtab** (Dependency used by messes.convert.mwTab_functions for conversion to the mwTab format accepted by Metabolomics Workbench)

## Examples

```
python3 -m pip install messes && coverage run -m pytest && coverage json -o coverage_report.json
```

## Evaluation signals

- A coverage_report.json is emitted containing per-module line/statement coverage for all five modules (messes.extract.extract, messes.validate.validate, messes.convert.convert, messes.convert.user_input_checking, messes.convert.mwTab_functions).
- The measured total coverage percentage exceeds the reported threshold of 90%.
- The same suite passes in the package's CI configuration across Linux, Windows, and MacOS environments as run by GitHub Actions.
- The validate command returns zero errors and warnings for JSON inputs that conform to the Experiment Description Specification plus a user-supplied --pds schema and the target format's built-in schema.
- Coverage of the validation and conversion modules (validate.validate, convert.user_input_checking, convert.mwTab_functions) is exercised through both valid and error/warning-raising inputs, not only the happy path.

## Limitations

- The conversion step requires the extracted JSON to be in the specific generalized MESSES schema; arbitrary schemas can be extracted but not converted, so format-conversion code coverage depends on conforming inputs.
- Coverage reported in the paper's CI context may differ from locally measured coverage if optional validation stages are skipped (the validate command can be skipped, though it is not recommended), leaving schema-checking paths unexercised.
- The validate command only flags errors and warnings against project parameters and schemas; passing validation does not guarantee semantic correctness of the underlying tabular data.

## Evidence

- [abstract] The package provides three commands, extract, validate, and convert, that implement a natural data transformation workflow.: "The package provides three commands, extract, validate, and convert, that implement a natural data transformation workflow."
- [readme] The MESSES package runs under Python 3.10+. Use pip_ to install.: [evidence span withheld — anchor status: external_doc]
- [readme] The validation step ensures the data that was extracted is valid against the `Experiment Description Specification: [evidence span withheld — anchor status: external_doc]
- [readme] Code coverage information: ":target: https://codecov.io/gh/MoseleyBioinformaticsLab/MESSES :alt: Code coverage information"
