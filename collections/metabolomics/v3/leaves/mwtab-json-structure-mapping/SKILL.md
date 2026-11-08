---
name: mwtab-json-structure-mapping
description: Use when a validated MESSES JSON representation of tagged experimental
  metadata (MS or NMR) must be transformed into mwTab JSON deposition format, using
  conversion directives with value_type str, matrix, or section.
when_to_use_negative:
- Input is raw, untagged Excel/CSV tabular data — run the MESSES extract command first
  to produce the JSONized intermediate
- The validated JSON has not passed the validate step against the Experiment Description
  Specification and format-specific schema — conversion relies on a specific schema
- Target format is anything other than mwTab, which is currently the only supported
  conversion format in MESSES
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3272
tools:
- name: MESSES
  role: Provides the convert command, conversion directives system (value_type str/matrix/section),
    and the messes.convert.mwtab_functions module that emits mwTab JSON
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: mwtab
  role: Target format module; messes.convert.mwtab_functions implements the conversion
    to the mwTab JSON deposition format
- name: jsonschema
  role: Used to structurally validate the converted JSON output against the deposition
    schema
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:35fba1ff8a3500f14e0c525f638208aef0f18f5f17cae99283d1540c6b5116f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/messes
  tools:
  - MESSES
  - jsonschema
  - mwtab
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# mwtab-json-structure-mapping

## Summary

Directive-driven conversion of a validated MESSES JSON intermediate into the mwTab JSON deposition format for Metabolomics Workbench, where each record declares a value_type (str, matrix, or section) that determines how it maps to output structure.

## When to use

Use when a validated MESSES JSON representation of tagged experimental metadata (MS or NMR) must be transformed into mwTab JSON deposition format, using conversion directives with value_type str, matrix, or section.

## When NOT to use

- Input is raw, untagged Excel/CSV tabular data — run the MESSES extract command first to produce the JSONized intermediate
- The validated JSON has not passed the validate step against the Experiment Description Specification and format-specific schema — conversion relies on a specific schema
- Target format is anything other than mwTab, which is currently the only supported conversion format in MESSES

## Inputs

- Validated MESSES JSON representation of tagged tabular (Excel/CSV) experimental metadata
- Conversion directives file declaring records with value_type fields ('str', 'matrix', 'section') and fields such as override, code, record_id, for_each, test, headers, collate, fields_to_headers
- Built-in or user-provided JSON schema specific to the target conversion format (e.g., mwTab deposition schema)

## Outputs

- mwTab JSON deposition structure accepted by Metabolomics Workbench
- jsonschema-validated converted JSON output

## How to apply

Load the validated MESSES JSON metadata and the mwTab conversion directives file, then implement the directive semantics: every record's 'value_type' field (one of 'str', 'matrix', 'section') determines which other fields are required and meaningful, resolved against the JSONized tables and records. Apply the recoding/addressing fields — 'override' to replace existing values, 'code' for programmatic transformations, and 'record_id' to target specific records — followed by the conditional/structural fields: 'for_each'/'test' iteration over records, 'headers' handling, 'collate' for merging entries, and 'fields_to_headers' for promoting data fields to table headers. Wire these semantics through the messes.convert.mwtab_functions module to emit the mwTab JSON structure. Only convert data that has passed the validate step against the Experiment Description Specification, any Protocol Dependent Schema, and the built-in schema for the conversion format, since conversion 'does rely on the data being in a specific schema'. Verify correctness by reproducing the worked directive examples from the conversion_directives and tutorial documentation.

## Related tools

- **MESSES** (Provides the convert command, conversion directives system (value_type str/matrix/section), and the messes.convert.mwtab_functions module that emits mwTab JSON) — https://github.com/MoseleyBioinformaticsLab/messes
- **mwtab** (Target format module; messes.convert.mwtab_functions implements the conversion to the mwTab JSON deposition format)
- **jsonschema** (Used to structurally validate the converted JSON output against the deposition schema)

## Examples

```
messes convert mwTab your_data.json your_format_data
```

## Evaluation signals

- Each worked example from the conversion_directives and tutorial documentation reproduces the documented mwTab output structure with matching keys and values
- The converted JSON passes jsonschema-based structural checks against the mwTab deposition schema
- The input JSON passed 'messes validate' with no errors or warnings before conversion was attempted
- Every record in the directives file declares a value_type of 'str', 'matrix', or 'section', and output fields vary consistently with that declared type
- The generated mwTab JSON is structurally acceptable for Metabolomics Workbench deposition

## Limitations

- Conversion from extracted JSON to another format relies on the data being in the specific MESSES Experiment Description Specification schema; arbitrary schemas will not convert
- Currently only mwTab is a supported conversion target format
- There is an initial steep learning curve; extraction, validation, and conversion settings must be worked out before the process is reliable
- The validate command can technically be skipped, but this is not recommended and risks converting invalid JSON
- The package requires Python 3.10+; on Mac OS an SSL certificate error may require running the Install Certificates.command shell script

## Evidence

- [methods] The convert command of MESSES supports converting JSON data to another JSON format or another supported format.: "The convert command of MESSES supports converting JSON data to another JSON format or another supported format."
- [methods] The validate command supports validating JSON data against schemas.: "The validate command of MESSES supports validating JSON data."
- [readme] Conversion to another format from the extracted JSON relies on the data being in a specific schema.: "Although any kind of data schema can be used for extraction into JSON, conversion 
to another format from the extracted JSON does rely on the data being in a specific 
schema."
- [readme] The validate command checks extracted data against the Experiment Description Specification, Protocol Dependent Schema, and a built-in schema for the conversion format.: [evidence span withheld — anchor status: external_doc]
- [readme] Currently supported formats: mwTab, used by the Metabolomics Workbench.: "Currently Supported Formats:
    
    * mwTab
        * Used by the `Metabolomics Workbench`_."
