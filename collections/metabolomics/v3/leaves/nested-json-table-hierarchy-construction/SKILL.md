---
name: nested-json-table-hierarchy-construction
description: 'Use when messy, arbitrarily organized Excel/CSV spreadsheet data must
  be turned into clean, well-structured JSONized metadata via a tag layer of #export,
  #automate, and #modify directives.'
when_to_use_negative:
- The extracted JSON already exists and the goal is format conversion to mwTab — use
  the convert command instead of re-running extraction
- The input is non-tabular (e.g., raw MS/NMR instrument binary files); MESSES was
  built for tabular data, only initially used to pull MS and NMR data into a database
- Data must be validated against the Experiment Description Specification or a Protocol
  Dependent Schema — use the validate command rather than re-implementing extraction
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MESSES
  role: Source package whose extract command and tagging system are being reconstructed;
    installed via pip into Python 3
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: Python 3
  role: Host language (Python 3.10+ per README; Python 3 per article) for re-implementing
    tag-layer parsing with pandas/openpyxl
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:f2da517aa65739eb2a13587ef0153b4108f993670a546f6e645ae8dd8fbac14b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/messes
  tools:
  - MESSES
  - Python 3
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# nested-json-table-hierarchy-construction

## Summary

Reconstructs MESSES's extract mechanism: a user-added layer of tags (#export, #automate, #modify) over messy tabular spreadsheet data drives transformation into a standardized, nested JSON metadata hierarchy suitable for Metabolomics Workbench depositions.

## When to use

Use when messy, arbitrarily organized Excel/CSV spreadsheet data must be turned into clean, well-structured JSONized metadata via a tag layer of #export, #automate, and #modify directives.

## When NOT to use

- The extracted JSON already exists and the goal is format conversion to mwTab — use the convert command instead of re-running extraction
- The input is non-tabular (e.g., raw MS/NMR instrument binary files); MESSES was built for tabular data, only initially used to pull MS and NMR data into a database
- Data must be validated against the Experiment Description Specification or a Protocol Dependent Schema — use the validate command rather than re-implementing extraction

## Inputs

- Tagged Excel (.xlsx) or CSV spreadsheet with an added tag layer of #export, #automate, and #modify directives
- Untagged raw tabular experimental data sections (data tables) referenced by the tag layer

## Outputs

- JSONized intermediate metadata file (e.g., your_data.json) organized into the six-table MESSES hierarchy (project, study, protocol, entity, measurement, factor)
- Rendered record lineages for extracted records

## How to apply

Load the tagged spreadsheets (Excel or CSV) containing the #export, #automate, and #modify directive sheets with pandas/openpyxl under Python 3. Re-implement parsing of the tag layer: #export directives define how JSON tables, records, and columns are constructed; #automate directives apply tags to untagged data automatically; #modify directives change names and values of data. Assemble the extracted records into the MESSES JSON hierarchy (project, study, protocol, entity, measurement, factor) exactly as the tags dictate. Support auxiliary behaviors such as modifying already-JSONized data and rendering record lineages. The rationale is that any tabular data can be transformed by adding a tag layer, making the extraction automatable and the resulting JSON interoperable and standardized.

## Related tools

- **MESSES** (Source package whose extract command and tagging system are being reconstructed; installed via pip into Python 3) — https://github.com/MoseleyBioinformaticsLab/messes
- **Python 3** (Host language (Python 3.10+ per README; Python 3 per article) for re-implementing tag-layer parsing with pandas/openpyxl)

## Examples

```
messes extract your_data.csv --output your_data.json
```

## Evaluation signals

- The produced JSON file exists and parses as valid JSON
- The produced JSON matches the reference JSON output from the MESSES examples directory (output_matches_reference)
- The output contains the six-table hierarchy: project, study, protocol, entity, measurement, factor, with keys dictated by the #export tags
- Running the extracted JSON through the validate command raises no errors against the Experiment Description Specification and any provided Protocol Dependent Schema
- Names and values changed by #modify directives appear in the JSON exactly as the directives dictate

## Limitations

- There is an initial steep learning curve; extraction, validation, and conversion settings must be worked out before the process fits a workflow
- Original data entry, manual tagging, and even automated tagging facilities can be messy, generating errors in the extracted JSONized representation
- Any schema can be extracted into JSON, but conversion to another format relies on the data being in the specific MESSES schema
- Requires Python 3.10+; on Mac OS an SSL certificate error may need fixing via Install Certificates.command

## Evidence

- [methods] The extract command of MESSES supports turning tabular data into JSON.: "The extract command of MESSES supports turning tabular data into JSON."
- [readme] we try to convert other people’s metadata messes into clean, well-structured, JSONized metadata: "We call it MESSES because we try to convert other people’s metadata messes into clean, well-structured, JSONized metadata."
- [readme] Simply add a layer of tags to any tabular data and MESSES can transform it into an intermediate JSON representation: "Simply add a layer of tags to any tabular data and 
MESSES can transform it into an intermediate JSON representation and then convert it to any of the supported formats."
- [readme] The extraction step adds a layer of tags to your raw tabular data, which may be automatable, and then extracts it into a JSONized form: "The extraction step adds a layer of tags 
to your raw tabular data, which may be automatable, and then extracts it into a JSONized form 
that it is more interoperable and more standardized."
- [readme] Although any kind of data schema can be used for extraction into JSON, conversion to another format from the extracted JSON does rely on the data being in a specific schema.: "Although any kind of data schema can be used for extraction into JSON, conversion 
to another format from the extracted JSON does rely on the data being in a specific schema."
