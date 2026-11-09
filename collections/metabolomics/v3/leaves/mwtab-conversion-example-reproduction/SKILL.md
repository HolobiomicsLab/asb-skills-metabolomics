---
name: mwtab-conversion-example-reproduction
description: Use when a validated MESSES JSON representation must be converted to
  mwTab JSON deposition format, or when reproduction of the documented conversion_directives
  examples is required to verify directive semantics.
when_to_use_negative:
- Input is raw untagged tabular data (CSV/Excel) that has not yet been extracted into
  the MESSES JSONized representation — run the extract (and validate) commands first
- The target repository format is not mwTab — MESSES currently supports only mwTab
  as an output format
- The MESSES JSON fails validation against the Experiment Description Specification,
  Protocol Dependent Schema, or format-specific schema — fix extraction errors before
  converting
edam_operation: http://edamontology.org/operation_0335
edam_topics: []
tools:
- name: MESSES
  role: Provides the extract, validate, and convert commands; its convert command
    implements the value_type-directed directive engine and messes.convert.mwtab_functions
    module that emit mwTab JSON
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: mwtab
  role: Target deposition format package for Metabolomics Workbench; messes.convert.mwtab_functions
    emits the mwTab JSON structure
- name: jsonschema
  role: Used for structural validation of the MESSES JSON and the converted output
    against the deposition schemas
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:9ab7d78a5f881fde44c7695e24ea28d1ec8f0258af4bced7237980996b696efd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/messes
  tools:
  - MESSES
  - jsonschema
  - mwtab
  edam_operation: http://edamontology.org/operation_0335
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mwtab-conversion-example-reproduction

## Summary

Reconstructs and verifies MESSES' convert command, whose value_type-directed directives (str, matrix, section) transform a validated MESSES JSON intermediate into mwTab JSON suitable for Metabolomics Workbench deposition.

## When to use

Use when a validated MESSES JSON representation must be converted to mwTab JSON deposition format, or when reproduction of the documented conversion_directives examples is required to verify directive semantics.

## When NOT to use

- Input is raw untagged tabular data (CSV/Excel) that has not yet been extracted into the MESSES JSONized representation — run the extract (and validate) commands first
- The target repository format is not mwTab — MESSES currently supports only mwTab as an output format
- The MESSES JSON fails validation against the Experiment Description Specification, Protocol Dependent Schema, or format-specific schema — fix extraction errors before converting

## Inputs

- Validated MESSES JSON representation of tabular experimental data (from the extract command)
- mwTab conversion directives file declaring value_type (str/matrix/section), override, code, record_id, for_each/test, headers, collate, and fields_to_headers fields
- Project JSON schemas: Experiment Description Specification, Protocol Dependent Schema, and format-specific schema

## Outputs

- mwTab JSON deposition structure accepted by the Metabolomics Workbench
- Per-example reproductions of the documented mwTab output structures (matching keys/values) for the worked conversion_directives and tutorial examples

## How to apply

Load the validated MESSES JSON and the mwTab conversion directives file with Python's json module, then implement the directive engine: every record must declare a 'value_type' field, and allowed value_types are 'str', 'matrix', and 'section' — the value_type determines which other fields are required and meaningful. Implement the recoding and addressing fields (override to replace existing values, code to apply programmatic transformations, record_id to target specific records) and the conditional/structural fields (for_each/test iteration over records, headers handling, collate for merging entries, fields_to_headers for promoting data fields to table headers). Wire the semantics through the messes.convert.mwtab_functions module to emit the target mwTab JSON structure, running conversions on the worked directive examples from the conversion_directives and tutorial documentation. Only convert data that has passed the validate step against the Experiment Description Specification, the Protocol Dependent Schema, and the built-in schema for the target format, as MESSES' conversion relies on the extracted JSON being in the specific schema.

## Related tools

- **MESSES** (Provides the extract, validate, and convert commands; its convert command implements the value_type-directed directive engine and messes.convert.mwtab_functions module that emit mwTab JSON) — https://github.com/MoseleyBioinformaticsLab/messes
- **mwtab** (Target deposition format package for Metabolomics Workbench; messes.convert.mwtab_functions emits the mwTab JSON structure)
- **jsonschema** (Used for structural validation of the MESSES JSON and the converted output against the deposition schemas)

## Examples

```
messes validate json your_data.json --pds your_schema.json --format desired_format && messes convert desired_format your_data.json your_format_data
```

## Evaluation signals

- Each worked documentation example from the conversion_directives and tutorial reproduces the documented mwTab output structure with matching keys/values
- Converted JSON passes jsonschema-based structural checks against the deposition schema
- Every converted record declares a valid value_type (one of str, matrix, section) and only the fields meaningful for that value_type
- Intermediate MESSES JSON passed the validate command (no errors) against the Experiment Description Specification, Protocol Dependent Schema, and mwTab format schema before conversion
- Override/code/record_id directive fields produce the expected value replacements and record targeting on the example records

## Limitations

- Conversion to another format relies on the extracted JSON being in the specific MESSES schema; arbitrary data schemas valid for extraction cannot necessarily be converted
- There is an initial steep learning curve to work out the extraction, validation, and conversion settings before the process can be added to routine workflows
- The validate command can be skipped, but this is not recommended; only mwTab is currently supported as a conversion target

## Evidence

- [full_text] allowed value_types are str, matrix, and section: [evidence span withheld — anchor status: not_found]
- [methods] convert command supports converting JSON data to another JSON format: "The convert command of MESSES supports converting JSON data to another JSON format or another supported format."
- [readme] validate command ensures data is valid before conversion: [evidence span withheld — anchor status: not_found]
- [readme] conversion relies on data being in a specific schema: [evidence span withheld — anchor status: external_doc]
- [methods] messes.convert.mwtab_functions module emits mwTab JSON: ".. automodule:: messes.convert.mwtab_functions
    :members:"
