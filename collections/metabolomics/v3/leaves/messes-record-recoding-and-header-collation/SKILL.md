---
name: messes-record-recoding-and-header-collation
description: Use when a validated MESSES JSONized metadata file must be transformed
  into mwTab JSON via conversion directives, and you need the value_type-driven semantics
  (str/matrix/section) plus recoding fields (override, code, record_id) and structural
  fields (for_each/test, headers, collate.
when_to_use_negative:
- Input is raw tabular data (Excel or CSV) that has not yet been tagged and extracted
  to JSON — use the extract command first.
- JSONized data has not been validated against the Experiment Description Specification,
  Protocol Dependent Schema, or a format-specific schema — run validate first, as
  skipping validation is not recommended.
- The target output format is not mwTab or another format explicitly supported by
  MESSES (mwTab is currently the only supported format).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: MESSES
  role: Provides the extract, validate, and convert commands and the messes.convert.mwtab_functions
    module that executes the conversion directives to emit mwTab JSON
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: Python 3
  role: Implementation language; json module loads the validated MESSES JSON and directives
    files, and the package runs under Python 3.10+
- name: jsonschema
  role: Used to structurally validate the converted JSON against the deposition schema
- name: mwtab
  role: Target deposition format/module wired through messes.convert.mwtab_functions
    to emit the Metabolomics Workbench mwTab JSON structure
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:14fbe454b9494bdbb1d487be40e1823770beb10faf24f2234d867c78244c92a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/messes
  tools:
  - MESSES
  - Python 3
  - jsonschema
  - mwtab
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# messes-record-recoding-and-header-collation

## Summary

Reconstructs the MESSES convert command's conversion-directive engine, where every record declares a value_type ('str', 'matrix', or 'section') that determines which recoding, addressing, and structural fields apply, to transform validated MESSES JSON into mwTab JSON depositions for the Metabolomics Workbench.

## When to use

Use when a validated MESSES JSONized metadata file must be transformed into mwTab JSON via conversion directives, and you need the value_type-driven semantics (str/matrix/section) plus recoding fields (override, code, record_id) and structural fields (for_each/test, headers, collate, fields_to_headers) implemented or reproduced.

## When NOT to use

- Input is raw tabular data (Excel or CSV) that has not yet been tagged and extracted to JSON — use the extract command first.
- JSONized data has not been validated against the Experiment Description Specification, Protocol Dependent Schema, or a format-specific schema — run validate first, as skipping validation is not recommended.
- The target output format is not mwTab or another format explicitly supported by MESSES (mwTab is currently the only supported format).

## Inputs

- Validated MESSES JSONized metadata (output of the extract/validate commands, conforming to the Experiment Description Specification and Protocol Dependent Schema)
- mwTab conversion directives file specifying value_type ('str', 'matrix', 'section') per record with override, code, record_id, for_each/test, headers, collate, and fields_to_headers fields
- Optional additional JSON schemas for format-specific validation (jsonschema format)

## Outputs

- mwTab JSON deposition file accepted by the Metabolomics Workbench
- jsonschema-based structural validation pass of the converted output against the deposition schema

## How to apply

Load the validated MESSES JSON and the mwTab conversion directives file with Python's json module, then implement value_type semantics so that each record's declared value_type ('str', 'matrix', or 'section') determines which other fields are required and meaningful when resolved against the JSONized tables and records. Implement recoding and addressing by applying 'override' to replace existing values, 'code' for programmatic transformations, and 'record_id' to target specific records. Implement conditional and structural behavior with 'for_each'/'test' iteration over records, 'headers' handling, 'collate' for merging entries, and 'fields_to_headers' for promoting data fields to table headers. Wire the semantics through the messes.convert.mwtab_functions module to emit the target mwTab JSON structure. This convert step is the third stage of the extract → validate → convert workflow, and it relies on the input already conforming to the Experiment Description Specification schema.

## Related tools

- **MESSES** (Provides the extract, validate, and convert commands and the messes.convert.mwtab_functions module that executes the conversion directives to emit mwTab JSON) — https://github.com/MoseleyBioinformaticsLab/messes
- **Python 3** (Implementation language; json module loads the validated MESSES JSON and directives files, and the package runs under Python 3.10+)
- **jsonschema** (Used to structurally validate the converted JSON against the deposition schema)
- **mwtab** (Target deposition format/module wired through messes.convert.mwtab_functions to emit the Metabolomics Workbench mwTab JSON structure)

## Examples

```
messes convert desired_format your_data.json your_format_data
```

## Evaluation signals

- Each worked example from the conversion_directives and tutorial documentation reproduces the documented mwTab output structure with matching keys and values.
- The converted JSON passes jsonschema-based structural checks against the deposition schema.
- Every record in the directives file declares a value_type of 'str', 'matrix', or 'section', and only fields meaningful for that value_type are consumed.
- The upstream JSON validated cleanly (no errors/warnings) against the Experiment Description Specification, Protocol Dependent Schema, and format-specific schema before conversion.
- 'fields_to_headers' directives result in data fields appearing as mwTab table headers, and 'collate' directives merge entries as expected in the output.

## Limitations

- There is an initial steep learning curve; extraction, validation, and conversion settings must be worked out before the process can be added to routine workflows.
- Conversion from extracted JSON requires the data to be in a specific schema (the MESSES Experiment Description Specification), even though extraction can target any schema.
- Manual tagging and automated tagging facilities can be messy, generating errors in the extracted JSONized representation that must be fixed via repeated validate cycles.
- Currently only mwTab (Metabolomics Workbench) is a supported conversion target.

## Evidence

- [methods] The convert command of MESSES supports converting JSON data to another JSON format or another supported format.: "The convert command of MESSES supports converting JSON data to another JSON format or another supported format."
- [readme] The conversion step converts the extracted data to the form that is accepted by the online repository.: [evidence span withheld — anchor status: external_doc]
- [readme] Although any kind of data schema can be used for extraction into JSON, conversion to another format from the extracted JSON does rely on the data being in a specific schema.: [evidence span withheld — anchor status: external_doc]
- [readme] But original data entry, manual tagging of tabular data, and even automated tagging facilities can be messy, generating errors in the extracted JSONized representation.: [evidence span withheld — anchor status: external_doc]
- [readme] Currently Supported Formats: mwTab Used by the Metabolomics Workbench.: "Currently Supported Formats:
    
    * mwTab
        * Used by the `Metabolomics Workbench`_."
