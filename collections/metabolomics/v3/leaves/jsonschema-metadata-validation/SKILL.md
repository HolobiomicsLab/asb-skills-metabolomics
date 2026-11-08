---
name: jsonschema-metadata-validation
description: Use when a MESSES 'extract' step has produced JSONized metadata from
  tagged tabular data (Excel/CSV) and it must be checked for errors and schema compliance
  before running 'convert' toward Metabolomics Workbench deposition.
when_to_use_negative:
- Input is still raw untagged tabular data (Excel/CSV) that has not been through the
  MESSES 'extract' command
- The data is already fully validated and the goal is format conversion itself, not
  schema checking
- Validating data for a repository format other than those MESSES supports (currently
  only mwTab for the Metabolomics Workbench)
tools:
- name: MESSES
  role: Provides the 'validate' command that checks extracted JSON against project
    and format-specific schemas
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: jsonschema
  role: Python library underlying the JSON schema validation checks performed by the
    validate command
- name: mwtab
  role: Target deposition format whose schema validated JSON must satisfy before conversion
    via messes.convert.mwtab_functions
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:ed0746f5ebbc47562166ebd2e87a385e62b9a204b734c74bedde27c9d2fbc5b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/messes
  tools:
  - MESSES
  - jsonschema
  - mwtab
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# jsonschema-metadata-validation

## Summary

Validates MESSES-extracted JSONized experimental metadata against the Experiment Description Specification, a Protocol Dependent Schema (PDS), user-supplied JSON schemas, and a format-specific deposition schema (e.g., mwTab) using the jsonschema library before conversion.

## When to use

Use when a MESSES 'extract' step has produced JSONized metadata from tagged tabular data (Excel/CSV) and it must be checked for errors and schema compliance before running 'convert' toward Metabolomics Workbench deposition.

## When NOT to use

- Input is still raw untagged tabular data (Excel/CSV) that has not been through the MESSES 'extract' command
- The data is already fully validated and the goal is format conversion itself, not schema checking
- Validating data for a repository format other than those MESSES supports (currently only mwTab for the Metabolomics Workbench)

## Inputs

- Extracted JSONized metadata from 'messes extract' (from tagged Excel or CSV tabular data)
- Protocol Dependent Schema JSON (--pds) defining project-specific parameters
- Optional additional user-supplied JSON schema
- Target format identifier (e.g., mwTab) selecting the built-in format-specific schema

## Outputs

- Validation report listing errors and warnings against the Experiment Description Specification, PDS, user schemas, and the format-specific (mwTab) schema
- Schema-valid JSON metadata approved for the 'convert' command

## How to apply

Run the MESSES 'validate' command on the extracted JSON, supplying a project-specific Protocol Dependent Schema via --pds and the target format via --format so the data is checked against the Experiment Description Specification, the PDS, any additional user-provided JSON schema, and a built-in schema for the target format. Iterate the extract-fix-validate cycle until no errors or warnings remain; the validate step can technically be skipped but the README states this is not recommended. Validation is also the final acceptance gate for the convert command's output: converted mwTab JSON should pass jsonschema-based structural checks against the deposition schema, and worked examples from the conversion_directives and tutorial documentation must reproduce their documented output structure (keys and values). The package is implemented in Python 3 (3.10+ per the README) and uses the jsonschema library (pypi.org/project/jsonschema) for the checks.

## Related tools

- **MESSES** (Provides the 'validate' command that checks extracted JSON against project and format-specific schemas) — https://github.com/MoseleyBioinformaticsLab/messes
- **jsonschema** (Python library underlying the JSON schema validation checks performed by the validate command)
- **mwtab** (Target deposition format whose schema validated JSON must satisfy before conversion via messes.convert.mwtab_functions)

## Examples

```
messes validate json your_data.json --pds your_schema.json --format desired_format
```

## Evaluation signals

- Zero errors and warnings reported by 'messes validate' after the extract-fix-validate iteration loop
- Converted output reproduces the documented mwTab structure (matching keys/values) for the worked examples in the conversion_directives and tutorial documentation
- Converted JSON passes jsonschema-based structural checks against the mwTab deposition schema
- The extraction, validation, and conversion settings are stable enough to be added routinely to data generation and analysis workflows

## Limitations

- Conversion to another format relies on the extracted JSON conforming to a specific schema (the MESSES Experiment Description Specification), so validation cannot rescue arbitrarily structured JSON
- Original data entry, manual tagging, and automated tagging can be messy and generate errors in the extracted JSONized representation, requiring repeated extract-fix-validate cycles
- There is an initial steep learning curve to work out the extraction, validation, and conversion settings

## Evidence

- [methods] The validate command of MESSES supports validating JSON data.: "The validate command of MESSES supports validating JSON data."
- [readme] validation against specification and schemas: [evidence span withheld — anchor status: not_found]
- [readme] additional schemas: [evidence span withheld — anchor status: not_found]
- [readme] validation is recommended before conversion: [evidence span withheld — anchor status: external_doc]
- [readme] iterative error-fixing loop: [evidence span withheld — anchor status: external_doc]
- [methods] jsonschema dependency: "(`jsonschema <https://pypi.org/project/jsonschema/>`_)"
- [readme] conversion depends on specific schema: [evidence span withheld — anchor status: external_doc]
