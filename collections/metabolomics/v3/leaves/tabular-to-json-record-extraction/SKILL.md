---
name: tabular-to-json-record-extraction
description: Use when messy tabular spreadsheet data (Excel or CSV) must be turned
  into key-based JSON records for a relational database or Metabolomics Workbench
  deposition, and a tag layer can be inserted into the sheet.
when_to_use_negative:
- The input is already a clean, validated JSON file — in that case use the MESSES
  validate or convert commands instead of extract
- The target output format is not supported — conversion from extracted JSON relies
  on the data being in a specific schema, and currently only mwTab (Metabolomics Workbench)
  is a supported format
- The data cannot accept an inserted tag layer (e.g., non-tabular or locked spreadsheets),
  since the tagging system is the key mechanism of extraction
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0091
tools:
- name: MESSES
  role: Core package whose extract command turns tabular data into JSON via the tagging
    system
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: Python 3
  role: Runtime/language in which MESSES is implemented (Python 3.10+ required); also
    usable as a library to extend extraction use-cases
- name: jsonschema
  role: Dependency used by MESSES validate to check extracted JSON against project
    schemas
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:5765ef509d9fdd096d2668ecf074a62d979aca99046a34fe0f49df478e99b290
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
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# tabular-to-json-record-extraction

## Summary

Transforms messy, arbitrarily organized tabular spreadsheet data (Excel or CSV) into clean, well-structured JSONized metadata by adding a layer of extraction tags (#export, #automate, #modify directives) to the spreadsheet and running MESSES's extract command. This is the first step of the MESSES extract-validate-convert workflow aimed at Metabolomics Workbench depositions.

## When to use

Use when messy tabular spreadsheet data (Excel or CSV) must be turned into key-based JSON records for a relational database or Metabolomics Workbench deposition, and a tag layer can be inserted into the sheet.

## When NOT to use

- The input is already a clean, validated JSON file — in that case use the MESSES validate or convert commands instead of extract
- The target output format is not supported — conversion from extracted JSON relies on the data being in a specific schema, and currently only mwTab (Metabolomics Workbench) is a supported format
- The data cannot accept an inserted tag layer (e.g., non-tabular or locked spreadsheets), since the tagging system is the key mechanism of extraction

## Inputs

- Tagged tabular spreadsheet (Excel or CSV) carrying #export, #automate, and #modify directive tags over the raw data tables
- Raw untagged tabular data (Excel or CSV) to which the tag layer is added

## Outputs

- JSONized metadata file organized into the six-table hierarchy: project, study, protocol, entity, measurement, factor
- Record lineages and modified already-JSONized data (auxiliary extract behaviors)

## How to apply

Insert an extra layer of tags at specific locations in the existing spreadsheet: #export directives define how JSON tables, records, and columns are constructed, #automate directives automatically apply tags to untagged data, and #modify directives change names and values of the data. Run the MESSES extract command on the tagged file to transform the tagged data sections (data tables) into JSON records, assembling the six-table hierarchy (project, study, protocol, entity, measurement, factor) exactly as the tags dictate. The extractor also supports auxiliary behaviors: modifying already-JSONized data and rendering record lineages. Then run messes validate on the output against the Experiment Description Specification, the Protocol Dependent Schema, any additional JSON schema, and a format-specific schema, fix errors and warnings in the original data, and repeat until no errors remain. Although any data schema can be used for extraction into JSON, note that conversion to another format relies on the data being in a specific schema.

## Related tools

- **MESSES** (Core package whose extract command turns tabular data into JSON via the tagging system) — https://github.com/MoseleyBioinformaticsLab/messes
- **Python 3** (Runtime/language in which MESSES is implemented (Python 3.10+ required); also usable as a library to extend extraction use-cases)
- **jsonschema** (Dependency used by MESSES validate to check extracted JSON against project schemas)

## Examples

```
messes extract your_data.csv --output your_data.json
```

## Evaluation signals

- Running the extractor on each tagged example spreadsheet produces a JSON file that exists, is valid JSON, and matches the reference JSON output from the MESSES examples directory (output_matches_reference)
- Extracted records conform to the Experiment Description Specification and the Protocol Dependent Schema when passed to messes validate (e.g., with --pds and --format flags)
- The output JSON contains the six-table hierarchy (project, study, protocol, entity, measurement, factor) exactly as the tag layer dictates
- The extract-validate-fix loop terminates with no remaining errors or warnings before convert is attempted
- #automate directives correctly propagated tags to previously untagged data sections (verifiable by diffing tag coverage against the tagged example spreadsheets)

## Limitations

- There is an initial steep learning curve; extraction, validation, and conversion settings must be worked out before the process fits into a routine workflow
- Original data entry, manual tagging of tabular data, and even automated tagging facilities can be messy, generating errors in the extracted JSONized representation — the validate step exists for this reason but can be (inadvisably) skipped
- Conversion to another format from the extracted JSON requires the data to be in a specific schema; only mwTab is currently supported as a target format
- On Mac OS an SSL certificate error may occur ('certificate verify failed: unable to get local issuer certificate'), fixed by running Install Certificates.command

## Evidence

- [abstract] The package provides three commands, extract, validate, and convert, that implement a natural data transformation workflow.: "The package provides three commands, extract, validate, and convert, that implement a natural data transformation workflow."
- [intro] We call it MESSES because we try to convert other people's metadata messes into clean, well-structured, JSONized metadata.: "We call it MESSES because we try to convert other people's metadata messes into clean, well-structured, JSONized metadata."
- [methods] The extract command of MESSES supports turning tabular data into JSON.: "The extract command of MESSES supports turning tabular data into JSON."
- [readme] The extraction step adds a layer of tags to your raw tabular data, which may be automatable, and then extracts it into a JSONized form: [evidence span withheld — anchor status: external_doc]
- [readme] The expected workflow is to use the "extract" command to transform your tabular data into JSON, then use the "validate" command: "The expected workflow is to use the "extract" command to transform your tabular data 
into JSON, then use the "validate" command to validate the JSON"
