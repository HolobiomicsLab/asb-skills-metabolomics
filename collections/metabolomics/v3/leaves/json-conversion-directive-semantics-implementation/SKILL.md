---
name: json-conversion-directive-semantics-implementation
description: 'Use when reconstructing or extending the MESSES conversion pipeline:
  the input is a validated MESSES JSON plus a conversion directives file, and the
  target is mwTab JSON for Metabolomics Workbench deposition.'
when_to_use_negative:
- Input is raw tabular data (Excel or CSV) that has not yet been extracted into the
  MESSES JSONized representation — use the extract command first
- The JSON has not been validated with messes validate (including --pds and --format
  checks) — the validate step should not be skipped
- The target format is not one of the currently supported formats (mwTab), or the
  goal is extraction of new metadata rather than conversion of existing JSON
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3314
- http://edamontology.org/topic_0219
tools:
- name: MESSES
  role: Source package whose convert command, messes.convert.mwtab_functions module,
    and directive semantics (value_type str/matrix/section, override, code, record_id,
    for_each/test, headers, collate, fields_to_headers) are being implemented
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: mwtab
  role: Target deposition format supported by the convert command; conversion is wired
    through messes.convert.mwtab_functions to emit mwTab JSON
- name: jsonschema
  role: Used for structural validation of the converted output against the deposition
    schema
  repo: https://pypi.org/project/jsonschema/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:eab53854fd427c4c76e8e35dce0300e143dd665a932715a44dfad4085ece6f90
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
  - http://edamontology.org/topic_3314
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# json-conversion-directive-semantics-implementation

## Summary

Implements the MESSES convert command's directive engine that transforms a validated MESSES JSON representation into mwTab JSON deposition format, where every record declares a value_type (str, matrix, or section) that determines which fields are required.

## When to use

Use when reconstructing or extending the MESSES conversion pipeline: the input is a validated MESSES JSON plus a conversion directives file, and the target is mwTab JSON for Metabolomics Workbench deposition.

## When NOT to use

- Input is raw tabular data (Excel or CSV) that has not yet been extracted into the MESSES JSONized representation — use the extract command first
- The JSON has not been validated with messes validate (including --pds and --format checks) — the validate step should not be skipped
- The target format is not one of the currently supported formats (mwTab), or the goal is extraction of new metadata rather than conversion of existing JSON

## Inputs

- Validated MESSES JSON intermediate representation (extracted tabular data conforming to the Experiment Description Specification)
- Conversion directives file (JSON) declaring value_type fields of str, matrix, or section
- mwTab conversion directives documentation and tutorial worked examples

## Outputs

- mwTab JSON deposition format file for Metabolomics Workbench
- Converted directive-example outputs matching the documented mwTab structures

## How to apply

Load the validated MESSES JSON metadata and the mwTab conversion directives file with Python's json module, then implement the value_type semantics so directive fields typed as str, matrix, or section are resolved against the JSONized tables and records. Implement the recoding and addressing fields — override to replace existing values, code to apply programmatic transformations, and record_id to target specific records — followed by the conditional and structural fields: for_each/test iteration over records, headers handling, collate for merging entries, and fields_to_headers for promoting data fields to table headers. Wire these semantics through the messes.convert.mwtab_functions module to emit the target mwTab JSON structure. Run the conversions on the worked directive examples from the conversion_directives and tutorial documentation, since MESSES only converts JSON that conforms to its Experiment Description Specification schema. Rationale: MESSES implements a three-command extract/validate/convert workflow, and the conversion step relies on data being in the specific MESSES schema, so directive semantics must be resolved against the validated intermediate JSON representation.

## Related tools

- **MESSES** (Source package whose convert command, messes.convert.mwtab_functions module, and directive semantics (value_type str/matrix/section, override, code, record_id, for_each/test, headers, collate, fields_to_headers) are being implemented) — https://github.com/MoseleyBioinformaticsLab/messes
- **mwtab** (Target deposition format supported by the convert command; conversion is wired through messes.convert.mwtab_functions to emit mwTab JSON)
- **jsonschema** (Used for structural validation of the converted output against the deposition schema) — https://pypi.org/project/jsonschema/

## Examples

```
messes validate json your_data.json --pds your_schema.json --format mwTab && messes convert mwTab your_data.json your_format_data
```

## Evaluation signals

- Each worked example from the conversion_directives and tutorial documentation reproduces the documented mwTab output structure (matching keys and values)
- Converted JSON passes jsonschema-based structural checks against the deposition schema
- The input JSON passed messes validate with no errors before conversion (validate/fix/repeat until no errors remain)
- Every output record correctly honors its declared value_type, with only the required and meaningful fields populated for str, matrix, or section
- override, code, record_id, for_each/test, collate, and fields_to_headers directives each produce the expected record-level transformations in the emitted mwTab JSON

## Limitations

- Conversion to another format from extracted JSON relies on the data being in the specific MESSES schema — arbitrary JSON schemas are not convertible
- Currently only mwTab is a supported conversion target (used by the Metabolomics Workbench)
- There is an initial steep learning curve; extraction, validation, and conversion settings must be worked out before the process integrates into data workflows
- The validate command can be skipped but is not recommended, and skipping it risks converting invalid JSONized representations produced by messy tagging

## Evidence

- [other] allowed value_types are str, matrix, and section; value determines which other fields are required: [evidence span withheld — anchor status: not_found]
- [methods] convert command supports converting JSON to another JSON format: "The convert command of MESSES supports converting JSON data to another JSON format or another supported format."
- [readme] conversion relies on data being in a specific schema: "Although any kind of data schema can be used for extraction into JSON, conversion 
to another format from the extracted JSON does rely on the data being in a specific 
schema."
- [abstract] three commands implement the transformation workflow: "The package provides three commands, extract, validate, and convert, that implement a natural data transformation workflow."
- [readme] mwtab is the Metabolomics Workbench format: "* mwTab
    * Used by the `Metabolomics Workbench`_."
