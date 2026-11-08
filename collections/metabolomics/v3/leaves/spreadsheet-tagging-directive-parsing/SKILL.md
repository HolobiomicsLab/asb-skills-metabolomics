---
name: spreadsheet-tagging-directive-parsing
description: Use when messy, arbitrarily organized tabular spreadsheet data (Excel
  or CSV) must be turned into structured, key-based JSON records by interpreting an
  inserted tagging layer of directives.
when_to_use_negative:
- The data is already in the final target format accepted by the repository (e.g.,
  mwTab) — the convert step, not extract/tag parsing, applies
- The goal is only to check an existing extracted JSON against the Experiment Description
  Specification or a project schema — the validate command handles this without re-parsing
  tags
- The input is not tabular spreadsheet data and cannot be organized into tagged data
  tables
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: MESSES
  role: Python package providing the extract command that parses the tag layer and
    turns tabular data into JSON
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: Python 3
  role: Implementation/runtime language in which MESSES and any re-implemented extractor
    run (Python 3.10+)
- name: jsonschema
  role: Dependency used by MESSES to validate the extracted JSON against the Experiment
    Description Specification and provided schemas
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:f6b3a3a2aa37ae0681fcde84d6799425776abafe157ef38708810d48c392e232
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# spreadsheet-tagging-directive-parsing

## Summary

Implements MESSES's tagging-based extraction mechanism: a layer of tags (#export, #automate, #modify directives) added to messy Excel/CSV spreadsheets drives transformation of data tables into key-based JSON records suitable for Metabolomics Workbench depositions.

## When to use

Use when messy, arbitrarily organized tabular spreadsheet data (Excel or CSV) must be turned into structured, key-based JSON records by interpreting an inserted tagging layer of directives.

## When NOT to use

- The data is already in the final target format accepted by the repository (e.g., mwTab) — the convert step, not extract/tag parsing, applies
- The goal is only to check an existing extracted JSON against the Experiment Description Specification or a project schema — the validate command handles this without re-parsing tags
- The input is not tabular spreadsheet data and cannot be organized into tagged data tables

## Inputs

- Tagged tabular spreadsheet data in Excel or CSV format with an added tag layer containing #export, #automate, and #modify directives
- Untagged raw tabular data that #automate directives can tag automatically
- Optionally, already-JSONized data to be modified by the extract command

## Outputs

- A JSONized intermediate representation of the tabular data organized into key-based records (project, study, protocol, entity, measurement, factor tables)
- Record lineage renderings for extracted records

## How to apply

Load the tagged spreadsheet (Excel or CSV) with pandas/openpyxl under Python 3 and parse the extra layer of tags the user inserted at specific locations on top of the raw data tables. Interpret #export directives that define how JSON tables, records, and columns are constructed, #automate directives that automatically apply tags to untagged data, and #modify directives that change names and values of data. Assemble the extracted records into the MESSES JSON hierarchy (project, study, protocol, entity, measurement, factor) exactly as the tags dictate, and support auxiliary behaviors such as modifying already-JSONized data and rendering record lineages. The rationale is that tags let MESSES transform arbitrary data sections into an interoperable, standardized intermediate representation that downstream validate and convert steps can consume against the Experiment Description Specification and a Protocol Dependent Schema.

## Related tools

- **MESSES** (Python package providing the extract command that parses the tag layer and turns tabular data into JSON) — https://github.com/MoseleyBioinformaticsLab/messes
- **Python 3** (Implementation/runtime language in which MESSES and any re-implemented extractor run (Python 3.10+))
- **jsonschema** (Dependency used by MESSES to validate the extracted JSON against the Experiment Description Specification and provided schemas)

## Examples

```
messes extract your_data.csv --output your_data.json
```

## Evaluation signals

- Running the extractor on each tagged example spreadsheet produces a JSON file that exists and parses as valid JSON
- The produced JSON matches the reference JSON output shipped in the MESSES examples directory
- The extracted JSON validates with 'messes validate json' against the Experiment Description Specification, the Protocol Dependent Schema, and any additional project schema without errors
- #modify directives are honored: changed names and values in the tags appear correctly in the extracted records
- #automate directives correctly propagate tags to previously untagged data sections

## Limitations

- There is an initial steep learning curve; tagging and extraction settings must be worked out before the process fits smoothly into data workflows
- Conversion from extracted JSON to another format relies on the data being in a specific schema — arbitrary JSON output cannot be converted
- Original data entry, manual tagging, and automated tagging can be messy and generate errors in the extracted JSONized representation, so the validate step should not be skipped
- MESSES requires Python 3.10+, and on Mac OS an SSL certificate error may need fixing via Install Certificates.command before installation/use

## Evidence

- [readme] tagging-based extraction via added tag layer: [evidence span withheld — anchor status: not_found]
- [readme] extraction step description: [evidence span withheld — anchor status: external_doc]
- [methods] extract command purpose: "The extract command of MESSES supports turning tabular data into JSON."
- [abstract] three-command workflow: "The package provides three commands, extract, validate, and convert, that implement a natural data transformation workflow."
- [readme] validation against schemas: [evidence span withheld — anchor status: not_found]
