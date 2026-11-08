---
name: mwtab-format-metadata-validation
description: Use when a tabular metabolomics dataset has already been extracted to
  MESSES JSON and must be checked for schema conformance before mwTab conversion and
  Metabolomics Workbench deposition.
when_to_use_negative:
- The input is still raw, untagged CSV/Excel data — it must first pass through the
  MESSES extract command to produce JSON
- The JSON is already validated and only the final mwTab conversion is needed (use
  the convert command instead)
- The goal is deposition to a repository other than Metabolomics Workbench, for which
  the built-in mwTab format schema does not apply
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: MESSES
  role: Provides the validate command that checks JSON data against the experiment
    description specification, optional PD schema, and built-in format-specific schema
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: jsonschema
  role: Python library to which MESSES delegates the core schema-conformance check
    of the validate command
  repo: https://pypi.org/project/jsonschema
- name: mwtab
  role: Related mwTab-format machinery in the MESSES convert module; the validated
    JSON targets the mwTab deposition format
  repo: https://pypi.org/project/messes
- name: Python 3
  role: Runtime for MESSES (Python 3.10+ per README) in which the validation step
    is executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:7cb3b82192031c3f6aa06a678f58509c9b95f893ac358c6832e3cd158ec1b315
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/messes
  tools:
  - MESSES
  - Python 3
  - jsonschema
  - mwtab
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# mwtab-format-metadata-validation

## Summary

Validates a MESSES-extracted JSON metadata representation against the Experiment Description Specification, an optional protocol-dependent (PD) schema, and a built-in mwTab format schema using the MESSES validate command (backed by the jsonschema library) before conversion to a Metabolomics Workbench deposition.

## When to use

Use when a tabular metabolomics dataset has already been extracted to MESSES JSON and must be checked for schema conformance before mwTab conversion and Metabolomics Workbench deposition.

## When NOT to use

- The input is still raw, untagged CSV/Excel data — it must first pass through the MESSES extract command to produce JSON
- The JSON is already validated and only the final mwTab conversion is needed (use the convert command instead)
- The goal is deposition to a repository other than Metabolomics Workbench, for which the built-in mwTab format schema does not apply

## Inputs

- MESSES JSON output produced by the extract command from tagged tabular data (CSV or Excel)
- Optional user-supplied JSON Schema file (e.g. a protocol-dependent/PD schema) passed via --pds
- Target format identifier for the built-in format-specific schema (mwTab)

## Outputs

- Validation verdict confirming (or refuting) conformance to the experiment description specification
- Warnings and errors emitted during validation, recorded in validation_report.txt

## How to apply

First obtain the JSONized metadata from the extract step, since the validate command operates on the MESSES JSON intermediate, not on raw CSV/Excel tables. Run the MESSES validate command in json mode, which by default validates the input with the Python jsonschema package against the integrated experiment_description_specification schema. For the paper's mwTab example data, supply the protocol-dependent (PD) schema via the --pds option and set the --format option to mwtab, which adds the built-in format-specific schema checks on top of the base specification. Capture any checks MESSES performs beyond what JSON Schema itself can express, and record all pass/fail verdicts, warnings, and errors in validation_report.txt. The expected workflow is to fix errors and warnings in the original tabular data, re-extract, and re-validate until no more errors remain; validation can be skipped but the project README explicitly recommends against it.

## Related tools

- **MESSES** (Provides the validate command that checks JSON data against the experiment description specification, optional PD schema, and built-in format-specific schema) — https://github.com/MoseleyBioinformaticsLab/messes
- **jsonschema** (Python library to which MESSES delegates the core schema-conformance check of the validate command) — https://pypi.org/project/jsonschema
- **mwtab** (Related mwTab-format machinery in the MESSES convert module; the validated JSON targets the mwTab deposition format) — https://pypi.org/project/messes
- **Python 3** (Runtime for MESSES (Python 3.10+ per README) in which the validation step is executed)

## Examples

```
messes validate json your_data.json --pds your_schema.json --format mwtab
```

## Evaluation signals

- The validate command completes and confirms the JSON conforms to the experiment description specification
- On the paper's mwTab-format example data, validation reproduces the expected warning/error feedback rather than a silent pass
- validation_report.txt is written and contains the pass/fail verdicts plus all warnings and errors
- Running with --pds and --format mwtab adds format-specific checks on top of the base jsonschema validation, as expected
- Re-extraction and re-validation after fixing errors terminates with zero errors, the state required before running convert

## Limitations

- There is an initial steep learning curve: data must be properly tagged and the tagging/experiment_description_specification documentation read before validation is meaningful
- The validate command can be skipped in the extract→validate→convert workflow, but this is explicitly not recommended by the README
- Conversion to another format relies on the extracted JSON being in the specific MESSES schema, so validation failures must be fixed upstream in the source data, not patched in the JSON

## Evidence

- [methods] validate command delegates check to jsonschema: "The validate command of MESSES supports validating JSON data."
- [abstract] three-command workflow: "The package provides three commands, extract, validate, and convert, that implement a natural data transformation workflow."
- [readme] validation step scope: [evidence span withheld — anchor status: external_doc]
- [readme] built-in format schema: "and a built 
in schema specific for the format you wish to convert to."
- [readme] iterate until no errors: "fix errors and warnings in the original data, repeat steps 1-3 until 
there are no more errors"
