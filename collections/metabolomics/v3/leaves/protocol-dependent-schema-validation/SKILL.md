---
name: protocol-dependent-schema-validation
description: Use when a MESSES-extracted JSON must be checked against the Protocol
  Dependent Schema and a target deposition format (mwTab) before running the convert
  step.
when_to_use_negative:
- The input is raw tabular data (Excel or CSV) that has not yet been extracted to
  MESSES JSON — use the extract command first.
- The data is already validated and ready for format conversion — the convert command,
  not validate, is the appropriate next step.
- The target repository format is not among MESSES's currently supported formats (currently
  only mwTab is supported).
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3071
- http://edamontology.org/topic_3174
tools:
- name: MESSES
  role: Provides the validate command that checks JSON against the experiment description
    specification, the PD schema, and format-specific schemas
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: jsonschema
  role: Python package that performs the core JSON Schema validation of the extracted
    JSON
- name: mwtab
  role: Supports the mwTab format-specific checks for Metabolomics Workbench depositions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:bc6bd0d4ad60c2829bea8fa3b7da2e469d0cde0ced513760cf174a7c1c11ca02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/messes
  tools:
  - MESSES
  - jsonschema
  - mwtab
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3071
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# protocol-dependent-schema-validation

## Summary

Validates a MESSES-extracted JSON representation against the Protocol Dependent Schema (PD schema) and a target format schema (e.g. mwTab) using the MESSES validate command, so that metadata errors are caught before conversion into a Metabolomics Workbench deposition format.

## When to use

Use when a MESSES-extracted JSON must be checked against the Protocol Dependent Schema and a target deposition format (mwTab) before running the convert step.

## When NOT to use

- The input is raw tabular data (Excel or CSV) that has not yet been extracted to MESSES JSON — use the extract command first.
- The data is already validated and ready for format conversion — the convert command, not validate, is the appropriate next step.
- The target repository format is not among MESSES's currently supported formats (currently only mwTab is supported).

## Inputs

- MESSES JSON file produced by the extract command (e.g. your_data.json)
- Protocol Dependent Schema as a JSON Schema file (e.g. your_schema.json, passed via --pds)
- Target format name for the format-specific schema check (e.g. mwTab)

## Outputs

- Validation verdict (JSON conforms to the experiment description specification and PD/format schemas)
- Validation report with warnings and errors (e.g. validation_report.txt)

## How to apply

Take the MESSES JSON emitted by the extract command and run the MESSES validate command, which by default validates the JSON with the jsonschema Python package against the integrated experiment_description_specification schema. To add format-specific checks, invoke validate with the mwTab format option and supply the Protocol Dependent Schema via the --pds option (e.g. a JSON Schema file for your project). The validation beyond what JSON Schema can express is performed internally by MESSES. Capture the verdicts plus any warnings and errors emitted during validation, then fix errors and warnings in the original tabular data and repeat extract/validate until no errors remain — the README notes validate can be skipped but is not recommended.

## Related tools

- **MESSES** (Provides the validate command that checks JSON against the experiment description specification, the PD schema, and format-specific schemas) — https://github.com/MoseleyBioinformaticsLab/messes
- **jsonschema** (Python package that performs the core JSON Schema validation of the extracted JSON)
- **mwtab** (Supports the mwTab format-specific checks for Metabolomics Workbench depositions)

## Examples

```
messes validate json your_data.json --pds your_schema.json --format mwtab
```

## Evaluation signals

- The validate command completes and confirms the JSON conforms to the experiment description specification.
- On the paper's mwTab-format example data, the expected warning/error feedback is reproduced.
- No errors or warnings remain in validation_report.txt after iterating on the source tabular data.
- The JSON validates against the supplied Protocol Dependent Schema file without jsonschema raising violations.

## Limitations

- Validation can be skipped, but the README states this is not recommended because extraction and tagging can generate errors in the JSONized representation.
- There is an initial steep learning curve to work out the extraction, validation, and conversion settings.
- Conversion to another format relies on the data being in a specific schema; the PD schema and format schema must match the intended target format (currently mwTab).

## Evidence

- [readme] validate with PD and format schema: [evidence span withheld — anchor status: external_doc]
- [readme] validate command purpose: [evidence span withheld — anchor status: external_doc]
- [methods] validate delegated to jsonschema: "The validate command of MESSES supports validating JSON data."
- [readme] example invocation in quickstart: [evidence span withheld — anchor status: external_doc]
- [readme] recommended workflow loop: [evidence span withheld — anchor status: external_doc]
