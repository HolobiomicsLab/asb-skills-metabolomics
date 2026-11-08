---
name: messes-json-validation-workflow-execution
description: Use when a JSONized metadata representation has been produced by the
  MESSES extract step from tagged CSV/Excel tabular data and must be checked against
  a JSON Schema before the convert step to a deposition format.
when_to_use_negative:
- The input is still raw tabular data (CSV/Excel) that has not been tagged and extracted
  into MESSES JSON by the extract command
- The goal is format conversion rather than conformance checking — use the MESSES
  convert command instead after validation succeeds
- The target output format is not among MESSES's supported formats (currently only
  mwTab)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: MESSES
  role: Provides the validate command that checks JSON data against the Experiment
    Description Specification, PD schema, and format-specific schema
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: jsonschema
  role: Python library to which the validate command's core JSON Schema conformance
    check is delegated
- name: mwtab
  role: Metabolomics Workbench format support for the mwTab format option in validate
- name: Python 3
  role: Runtime environment (Python 3.10+) in which the MESSES package and its validate
    command execute
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:6e2816a4aa501790099ad0f65b1bcc805798266fff5a463bdb1a0d6de631709b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/messes
  tools:
  - MESSES
  - Python 3
  - jsonschema
  - mwtab
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# messes-json-validation-workflow-execution

## Summary

Run the MESSES validate command to check that JSON extracted from tagged tabular data conforms to the Experiment Description Specification and, for Metabolomics Workbench submissions, the mwTab-format and Protocol Dependent (PD) schemas before conversion.

## When to use

Use when a JSONized metadata representation has been produced by the MESSES extract step from tagged CSV/Excel tabular data and must be checked against a JSON Schema before the convert step to a deposition format.

## When NOT to use

- The input is still raw tabular data (CSV/Excel) that has not been tagged and extracted into MESSES JSON by the extract command
- The goal is format conversion rather than conformance checking — use the MESSES convert command instead after validation succeeds
- The target output format is not among MESSES's supported formats (currently only mwTab)

## Inputs

- MESSES JSON output produced by the extract command from tagged CSV or Excel tabular data
- Protocol Dependent (PD) schema JSON file (e.g. your_schema.json) supplied via --pds
- Target format name (currently supported: mwTab, used by the Metabolomics Workbench)

## Outputs

- Pass/fail validation verdict confirming the JSON conforms to the Experiment Description Specification and format-specific schema
- Warnings and errors emitted during validation (e.g. recorded in validation_report.txt)

## How to apply

Load the MESSES JSON emitted by the extract command, then invoke 'messes validate json' on it; the core check is delegated to the jsonschema Python library validating against the integrated experiment_description_specification schema. For mwTab-format targets such as Metabolomics Workbench submissions, add the --format mwtab option and supply a Protocol Dependent Schema with --pds to layer format-specific checks on top of the base specification. Capture both JSON Schema-level verdicts and the additional validation MESSES performs internally beyond JSON Schema's capabilities, since original tagging can generate errors in the extracted JSONized representation. Record the pass/fail verdict plus all warnings and errors emitted; if errors are found, fix the original tagged data, re-extract, and repeat until no errors remain (the validate step can be skipped but is not recommended).

## Related tools

- **MESSES** (Provides the validate command that checks JSON data against the Experiment Description Specification, PD schema, and format-specific schema) — https://github.com/MoseleyBioinformaticsLab/messes
- **jsonschema** (Python library to which the validate command's core JSON Schema conformance check is delegated)
- **mwtab** (Metabolomics Workbench format support for the mwTab format option in validate)
- **Python 3** (Runtime environment (Python 3.10+) in which the MESSES package and its validate command execute)

## Examples

```
messes validate json your_data.json --pds your_schema.json --format mwtab
```

## Evaluation signals

- The validate command completes and confirms the JSON conforms to the experiment description specification
- For the paper's mwTab example data, validation reproduces the expected warning/error feedback
- No JSON Schema errors remain after iterating the extract→validate fix loop on the original tagged data
- Format-specific PD schema and mwTab-format checks pass when --pds and --format mwtab are supplied
- All warnings and errors from validation are captured in the validation report rather than silently dropped

## Limitations

- Validation against arbitrary schemas depends on the data being in a specific schema — conversion from extracted JSON relies on the MESSES Experiment Description Specification
- There is an initial steep learning curve: tagging and the Experiment Description Specification must be understood before validate can be applied meaningfully
- The validate command can be skipped, but this is not recommended because messy original data entry or tagging generates errors in the extracted JSONized representation
- JSON Schema alone is insufficient — MESSES performs additional validation internally beyond JSON Schema's capabilities

## Evidence

- [methods] validate command core check: [evidence span withheld — anchor status: not_found]
- [methods] validate command purpose: "The validate command of MESSES supports validating JSON data."
- [readme] README validation scope: [evidence span withheld — anchor status: external_doc]
- [readme] README workflow rationale: [evidence span withheld — anchor status: external_doc]
- [readme] README quickstart commands: [evidence span withheld — anchor status: external_doc]
