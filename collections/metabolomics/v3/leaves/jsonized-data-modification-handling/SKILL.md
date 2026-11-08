---
name: jsonized-data-modification-handling
description: 'Use when a tagged spreadsheet extraction must apply #modify directives
  to rename or revalue data, or when already-produced MESSES JSON records need name/value
  modification or lineage rendering rather than a fresh extraction.'
when_to_use_negative:
- 'When the goal is only to construct the JSON table/record/column structure from
  unmodified data — that is the #export directive behavior, not modification'
- When the data is already clean, structured JSON that needs no name/value changes
  — go directly to messes validate/convert
- When converting validated JSON to a final format such as mwTab — that is the convert
  command's job, not the modify/extract step
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3077
- http://edamontology.org/topic_3489
tools:
- name: MESSES
  role: 'Python package whose extract command implements the tagging system, including
    #modify directives for changing names and values and lineage rendering of JSONized
    records'
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: Python 3
  role: Implementation language used to load tagged spreadsheets (pandas/openpyxl)
    and re-implement or run the modify/lineage behaviors
- name: jsonschema
  role: Underlying validation dependency used by messes validate to check modified
    JSON against the Experiment Description Specification and user schemas
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:d1440ad17982e214bc7e047d5caf21f1f912c89beb87c0430bad8ff01e2e3b3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/messes
  tools:
  - MESSES
  - Python 3
  - jsonschema
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3077
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# jsonized-data-modification-handling

## Summary

Reconstructs MESSES's auxiliary extract behaviors for modifying data after JSONization: applying #modify directives that change names and values of data and rendering record lineages for the extracted JSON records.

## When to use

Use when a tagged spreadsheet extraction must apply #modify directives to rename or revalue data, or when already-produced MESSES JSON records need name/value modification or lineage rendering rather than a fresh extraction.

## When NOT to use

- When the goal is only to construct the JSON table/record/column structure from unmodified data — that is the #export directive behavior, not modification
- When the data is already clean, structured JSON that needs no name/value changes — go directly to messes validate/convert
- When converting validated JSON to a final format such as mwTab — that is the convert command's job, not the modify/extract step

## Inputs

- Tagged Excel or CSV spreadsheet containing #modify (and optionally #export/#automate) directive rows
- Already-JSONized MESSES extract output (intermediate JSON representation)
- Optional JSON schema files for validation (e.g., project schema, Protocol Dependent Schema)

## Outputs

- Modified MESSES JSON records with altered names and values as dictated by #modify directives
- Record lineage renderings showing provenance of extracted records
- Validation reports from messes validate

## How to apply

Parse the tag layer of the tagged spreadsheet, paying special attention to #modify directives, which change names and values of the underlying data (as distinct from #export directives that define JSON table/record/column construction and #automate directives that auto-tag untagged data). Apply the modify operations during the extract step so that the resulting JSONized records reflect the changed names/values, assembling them into the six-table MESSES hierarchy (project, study, protocol, entity, measurement, factor). If the input is already JSONized data rather than a raw spreadsheet, apply the same modify behavior directly on the JSON records and render record lineages so the provenance of each extracted record is traceable. Validate the resulting JSON against the Experiment Description Specification and any Protocol Dependent Schema using messes validate before considering the modification correct.

## Related tools

- **MESSES** (Python package whose extract command implements the tagging system, including #modify directives for changing names and values and lineage rendering of JSONized records) — https://github.com/MoseleyBioinformaticsLab/messes
- **Python 3** (Implementation language used to load tagged spreadsheets (pandas/openpyxl) and re-implement or run the modify/lineage behaviors)
- **jsonschema** (Underlying validation dependency used by messes validate to check modified JSON against the Experiment Description Specification and user schemas)

## Examples

```
messes extract your_data.csv --output your_data.json
```

## Evaluation signals

- Modified records reflect exactly the name/value changes dictated by the #modify directive rows in the tag layer
- Output JSON is parseable and conforms to the six-table MESSES hierarchy (project, study, protocol, entity, measurement, factor)
- Rendered record lineages trace each extracted record back to its source tagged spreadsheet location
- messes validate on the modified JSON reports no errors against the Experiment Description Specification and any provided schema
- Extract output matches the reference JSON files in the MESSES examples/figshare dataset when run on documented example spreadsheets

## Limitations

- Manual tagging of tabular data, and even automated tagging facilities, can be messy and generate errors in the extracted JSONized representation, requiring iterative validate-and-fix cycles
- There is an initial steep learning curve; modify behavior only works within the tagging/extract system after the tagging documentation is understood
- Conversion to another format from extracted JSON relies on the data being in a specific schema, so modifications must keep data within the MESSES Experiment Description Specification

## Evidence

- [other] modify directives change names and values of data: [evidence span withheld — anchor status: not_found]
- [other] auxiliary extract behaviors include modifying JSONized data and lineage rendering: [evidence span withheld — anchor status: not_found]
- [readme] The key to MESSES is the tagging system applied to tabular data: [evidence span withheld — anchor status: not_found]
- [readme] validate step guards against messy tagging errors: [evidence span withheld — anchor status: external_doc]
- [methods] extract command supports turning tabular data into JSON: "The extract command of MESSES supports turning tabular data into JSON."
