---
name: json-schema-validated-metadata-checking
description: Use when a tabular-data-derived JSON metadata file must be checked for
  conformance to the MESSES experiment_description_specification (and optionally a
  Protocol Dependent Schema or target format schema) before conversion to a deposition
  format.
when_to_use_negative:
- The input has not yet been extracted — raw tabular data (CSV/Excel) still needs
  the MESSES extract command's tagging system before it can be validated as JSON
- The validated JSON is already error-free and the goal is format conversion, not
  checking — go directly to the MESSES convert command
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3071
- http://edamontology.org/topic_3172
tools:
- name: MESSES
  role: Provides the validate command that runs JSON Schema-based conformance checking
    of extracted metadata against the experiment_description_specification, optional
    PDS, and target-format schemas
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: jsonschema
  role: Python library that performs the core JSON Schema validation delegated to
    by the MESSES validate command
- name: mwtab
  role: Supports the mwTab-format-specific schema checks invoked via the validate
    command's --format option for Metabolomics Workbench deposition
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:a51dca4cc4853afccf10f6c97fb20b50d9fd0a193a3cea6803fd98ba42ce57fa
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
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# json-schema-validated-metadata-checking

## Summary

Validates an extracted MESSES JSONized metadata representation against the Experiment Description Specification (via the jsonschema library), optionally adding the Protocol Dependent Schema (PDS) and format-specific (mwTab) schema checks before conversion.

## When to use

Use when a tabular-data-derived JSON metadata file must be checked for conformance to the MESSES experiment_description_specification (and optionally a Protocol Dependent Schema or target format schema) before conversion to a deposition format.

## When NOT to use

- The input has not yet been extracted — raw tabular data (CSV/Excel) still needs the MESSES extract command's tagging system before it can be validated as JSON
- The validated JSON is already error-free and the goal is format conversion, not checking — go directly to the MESSES convert command

## Inputs

- MESSES-generated JSON metadata file produced by the extract command from tabular (CSV/Excel) data
- Optional Protocol Dependent Schema (PDS) JSON schema file
- Optional additional user-supplied JSON Schema file

## Outputs

- Pass/fail validation verdict confirming (or refuting) conformance to the experiment_description_specification
- Validation warnings and errors (optionally captured in validation_report.txt)

## How to apply

After the MESSES extract command has turned tabular data (CSV or Excel) into JSON, run the MESSES validate command, which delegates the core check to the jsonschema Python package against the integrated experiment_description_specification schema. Optionally supply a Protocol Dependent Schema via the --pds option and a target format (e.g. mwTab) via the --format option to add schema-specific checks beyond the base specification. Iterate on the source tabular data to fix errors and warnings, repeating extract–validate until no more errors are emitted. Success is judged by validate completing and confirming the JSON conforms to the experiment description specification; verdicts, warnings, and errors can be captured in a validation report. Validation can technically be skipped, but the README states it is not recommended because conversion relies on the data being in the specific MESSES schema.

## Related tools

- **MESSES** (Provides the validate command that runs JSON Schema-based conformance checking of extracted metadata against the experiment_description_specification, optional PDS, and target-format schemas) — https://github.com/MoseleyBioinformaticsLab/messes
- **jsonschema** (Python library that performs the core JSON Schema validation delegated to by the MESSES validate command)
- **mwtab** (Supports the mwTab-format-specific schema checks invoked via the validate command's --format option for Metabolomics Workbench deposition)

## Examples

```
messes validate json your_data.json --pds your_schema.json --format mwtab
```

## Evaluation signals

- validate command completes and explicitly confirms the JSON conforms to the experiment description specification
- The extracted JSON passes jsonschema validation against the integrated MESSES schema (and, for mwTab examples, against the PDS and mwTab format schema)
- For the paper's mwTab example data, validation reproduces the expected warning/error feedback
- Repeated extract–validate cycles terminate with no remaining errors before running convert
- Verdicts, warnings, and errors are recorded (e.g. in validation_report.txt)

## Limitations

- The validate command can be skipped, but the README states this is not recommended since conversion to another format relies on the data being in the specific MESSES schema
- There is an initial steep learning curve: data must first be properly tagged and the Experiment Description Specification understood before validation is meaningful
- JSON Schema alone cannot capture all checks — MESSES performs additional validation beyond JSON Schema's capabilities internally

## Evidence

- [methods] validate delegates to jsonschema: "The validate command of MESSES supports validating JSON data."
- [methods] jsonschema dependency: "(`jsonschema <https://pypi.org/project/jsonschema/>`_)"
- [readme] validation step purpose: [evidence span withheld — anchor status: external_doc]
- [readme] PDS and format schemas: [evidence span withheld — anchor status: external_doc]
- [readme] conversion depends on schema: "conversion 

to another format from the extracted JSON does rely on the data being in a specific
schema"
- [readme] example validate invocation: [evidence span withheld — anchor status: external_doc]
- [readme] iterate until no errors: "use the "validate" command to validate the JSON based on your specific 
project schema, fix errors and warnings in the original data, repeat steps 1-3 until 
there are no more errors"
