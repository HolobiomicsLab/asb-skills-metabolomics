---
name: metabolomics-workbench-experiment-description-conformance
description: Use when tabular data has already been extracted into MESSES JSON and
  must be checked for conformance to the Experiment Description Specification and
  PD schema before mwTab conversion.
when_to_use_negative:
- The input is still raw tabular data (CSV/Excel) that has not been through the MESSES
  extract step — run extract first to obtain the JSONized representation.
- The data is already validated JSON ready for the convert command, or the target
  repository format is not one of MESSES's supported formats (currently only mwTab).
- The data cannot be tagged per the MESSES tagging system, since extraction into the
  Experiment Description Specification schema is a prerequisite for validation.
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3521
tools:
- name: MESSES
  role: Provides the validate command that checks extracted JSON against the Experiment
    Description Specification, PD schema, and format-specific schema
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: jsonschema
  role: Python library to which the core schema-conformance check of the validate
    command is delegated
- name: mwtab
  role: Associated package for the mwTab deposition format whose format-specific schema
    is checked during validation
- name: Python 3
  role: Runtime for the MESSES package (Python 3.10+ per the README) and its validation
    dependencies
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:ce869fc0aef7e1f996a22808447241cb8efccf8fea645c7850ef5b8deeb05852
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
  - http://edamontology.org/topic_3521
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metabolomics-workbench-experiment-description-conformance

## Summary

Validates a MESSES-extracted JSON metadata representation against the Experiment Description Specification (via the jsonschema library) plus format-specific and Protocol Dependent (PD) schema checks, so the JSON is fit for conversion into a Metabolomics Workbench mwTab deposition.

## When to use

Use when tabular data has already been extracted into MESSES JSON and must be checked for conformance to the Experiment Description Specification and PD schema before mwTab conversion.

## When NOT to use

- The input is still raw tabular data (CSV/Excel) that has not been through the MESSES extract step — run extract first to obtain the JSONized representation.
- The data is already validated JSON ready for the convert command, or the target repository format is not one of MESSES's supported formats (currently only mwTab).
- The data cannot be tagged per the MESSES tagging system, since extraction into the Experiment Description Specification schema is a prerequisite for validation.

## Inputs

- MESSES JSON output from the extract step (JSONized intermediate representation of tagged tabular data)
- Optional user-provided JSON schema (--pds / Protocol Dependent Schema)
- Target format name for the format-specific schema check (e.g. mwTab)

## Outputs

- Validation pass/fail verdict on conformance to the experiment description specification
- Warnings and errors emitted during validation (recordable in validation_report.txt)

## How to apply

Run the MESSES validate command on the JSON emitted by the extract step; by default it validates via the jsonschema Python package against the integrated experiment_description_specification schema. Add format-specific and protocol-dependent checks by passing the mwTab format option and the PD schema option (--pds) for the paper's mwTab example. Beyond JSON Schema's intrinsic capabilities, capture the additional validation MESSES performs internally, since manual tagging and automated tagging can inject errors into the extracted JSONized representation. Record pass/fail verdicts plus any warnings and errors (e.g. in validation_report.txt), fix errors in the original tagged data, and repeat the extract-validate cycle until no errors remain before invoking convert.

## Related tools

- **MESSES** (Provides the validate command that checks extracted JSON against the Experiment Description Specification, PD schema, and format-specific schema) — https://github.com/MoseleyBioinformaticsLab/messes
- **jsonschema** (Python library to which the core schema-conformance check of the validate command is delegated)
- **mwtab** (Associated package for the mwTab deposition format whose format-specific schema is checked during validation)
- **Python 3** (Runtime for the MESSES package (Python 3.10+ per the README) and its validation dependencies)

## Examples

```
messes validate json your_data.json --pds your_schema.json --format desired_format
```

## Evaluation signals

- The validate command completes and confirms the JSON conforms to the experiment description specification.
- Pass/fail verdicts plus any warnings and errors emitted during validation are recorded (e.g. in validation_report.txt).
- JSON Schema validation via the jsonschema library raises no unresolved errors against the experiment_description_specification schema.
- With the mwTab format option and PD schema option supplied, no format-specific or protocol-dependent errors remain.
- On the paper's mwTab-format example data, the run reproduces the expected warning/error feedback.

## Limitations

- The validate command can be skipped, but the README explicitly states it is not recommended; skipping risks non-conformant JSON reaching convert.
- There is an initial steep learning curve: data must be tagged per the MESSES tagging system and the Experiment Description Specification before validation is meaningful.
- Original data entry, manual tagging, and even automated tagging facilities can be messy, generating errors in the extracted JSONized representation that validation must catch iteratively.
- Conversion to another format relies on the data being in a specific schema; a validated-but-nonstandard schema may still fail at the convert step.

## Evidence

- [methods] validate command delegates to jsonschema: "The validate command of MESSES supports validating JSON data."
- [abstract] three-command workflow: "The package provides three commands, extract, validate, and convert, that implement a natural data transformation workflow."
- [readme] validation scope per README: [evidence span withheld — anchor status: not_found]
- [readme] why validate exists: [evidence span withheld — anchor status: external_doc]
- [readme] iterate until clean: [evidence span withheld — anchor status: external_doc]
