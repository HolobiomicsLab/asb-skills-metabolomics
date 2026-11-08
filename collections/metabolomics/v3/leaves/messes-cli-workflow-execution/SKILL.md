---
name: messes-cli-workflow-execution
description: Use when a metabolomics dataset exists as messy tabular spreadsheets
  (Excel/CSV) tagged per the MESSES tagging system and must be transformed into Metabolomics
  Workbench mwTab format via the extract-validate-convert pipeline.
when_to_use_negative:
- Input is already valid, structured JSON — the extract step targets raw tagged tabular
  (Excel/CSV) data, not JSONized inputs
- The target repository format is not mwTab — currently mwTab (Metabolomics Workbench)
  is the supported conversion format
- Data lacks the MESSES tagging layer — skipping tagging means the commands cannot
  transform untagged tables
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: MESSES
  role: Python package providing the extract, validate, and convert CLI commands that
    implement the tabular-to-mwTab transformation workflow
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: jsonschema
  role: Underlying validation library used by the MESSES validate command to check
    extracted JSON against the specification and user schemas
- name: mwtab
  role: Target format; messes.convert.mwtab_functions performs the conversion into
    mwTab JSON for Metabolomics Workbench deposition
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:5acd993ad0b56f9e6179d30cb6b547a9c54e3df909119ce69eafa514ad5f6f1e
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
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# messes-cli-workflow-execution

## Summary

Execute the MESSES three-command workflow — extract, validate, convert — to transform tagged tabular metabolomics spreadsheets into deposition-ready mwTab JSON and tab-delimited outputs for Metabolomics Workbench submissions.

## When to use

Use when a metabolomics dataset exists as messy tabular spreadsheets (Excel/CSV) tagged per the MESSES tagging system and must be transformed into Metabolomics Workbench mwTab format via the extract-validate-convert pipeline.

## When NOT to use

- Input is already valid, structured JSON — the extract step targets raw tagged tabular (Excel/CSV) data, not JSONized inputs
- The target repository format is not mwTab — currently mwTab (Metabolomics Workbench) is the supported conversion format
- Data lacks the MESSES tagging layer — skipping tagging means the commands cannot transform untagged tables

## Inputs

- Tagged tabular spreadsheet data (Excel or CSV) with MESSES tagging layer
- User-supplied JSON schema for validation (--pds)
- Tagged IC-FTMS mouse colon tissue example dataset from Metabolomics Workbench Study ST001447 (in the MESSES examples/tutorial directory)
- mwTab conversion directives specifying the target format

## Outputs

- JSONized metadata tables and records from extract (e.g. your_data.json)
- Validation report of errors and warnings against the experiment-description specification
- mwTab JSON file (e.g. output.json)
- mwTab tab-delimited file (e.g. output.txt)

## How to apply

Install MESSES under Python 3.10+ (pip install messes) and confirm the data carries the MESSES tagging layer described in the documentation; untagged tables must be tagged first or the commands cannot transform them. Run 'messes extract' on the tagged CSV/Excel file to produce a JSONized intermediate metadata representation, as demonstrated on the worked IC-FTMS mouse colon tissue example from Metabolomics Workbench Study ST001447. Next run 'messes validate json' with the default experiment-description specification plus the Protocol Dependent Schema (--pds), any user-supplied JSON schema, and the target format schema (--format desired_format), which checks the JSON via the jsonschema package; fix errors and warnings in the original spreadsheet and repeat extract-validate until no errors remain. Finally run 'messes convert' with mwTab conversion directives to emit the validated JSON as mwTab format. Verification: both a mwTab JSON file (e.g. output.json) and a mwTab tab-delimited file (e.g. output.txt) are produced; check they exist and conform to their expected formats.

## Related tools

- **MESSES** (Python package providing the extract, validate, and convert CLI commands that implement the tabular-to-mwTab transformation workflow) — https://github.com/MoseleyBioinformaticsLab/messes
- **jsonschema** (Underlying validation library used by the MESSES validate command to check extracted JSON against the specification and user schemas)
- **mwtab** (Target format; messes.convert.mwtab_functions performs the conversion into mwTab JSON for Metabolomics Workbench deposition)

## Examples

```
messes extract your_data.csv --output your_data.json && messes validate json your_data.json --pds your_schema.json --format mwTab && messes convert mwTab your_data.json your_format_data
```

## Evaluation signals

- Running the full workflow on the ST001447 IC-FTMS example produces exactly two artifacts: a mwTab JSON file (output.json) and a mwTab tab-delimited file (output.txt)
- messes validate exits with no errors and warnings after schema checking against the experiment-description specification and --pds schema
- The mwTab JSON output parses as valid JSON and conforms to the mwTab schema
- The tab-delimited mwTab file contains the expected mwTab sections matching the converted records
- Extracted intermediate JSON is valid against the Experiment Description Specification before conversion is attempted

## Limitations

- Validation can be skipped, but this is not recommended; unvalidated JSON may fail conversion
- There is an initial steep learning curve to work out extraction, validation, and conversion settings
- Manual tagging of tabular data can be messy, generating errors in the extracted JSONized representation that require repeated extract-validate cycles
- Conversion to another format relies on the data being in the specific MESSES Experiment Description schema; the package runs under Python 3.10+

## Evidence

- [abstract] The package provides three commands, extract, validate, and convert, that implement a natural data transformation workflow.: "The package provides three commands, extract, validate, and convert, that implement a natural data transformation workflow."
- [results] extract-validate-convert on IC-FTMS ST001447 example produces output.json and output.txt: [evidence span withheld — anchor status: not_found]
- [readme] messes extract / validate / convert CLI sequence: [evidence span withheld — anchor status: not_found]
- [readme] MESSES runs under Python 3.10+: [evidence span withheld — anchor status: external_doc]
