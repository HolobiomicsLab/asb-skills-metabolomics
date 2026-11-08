---
name: mwtab-format-conversion
description: Use when a messy tabular metabolomics dataset (Excel/CSV, e.g. MS or
  NMR experiment metadata) must be transformed into mwTab format for Metabolomics
  Workbench deposition after MESSES extract and validate steps.
when_to_use_negative:
- The input is already in mwTab format or another repository-accepted submission format
  — MESSES convert targets tabular-to-JSON-to-mwTab transformation, not re-validation
  of finished deposits.
- The data cannot be tagged into the Experiment Description Specification schema —
  conversion to mwTab from extracted JSON relies on the data being in that specific
  schema.
- The target repository format is not mwTab — mwTab is currently the only supported
  conversion format in MESSES.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MESSES
  role: Python package providing the extract, validate, and convert commands used
    to transform tagged tabular data into mwTab deposition files
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: jsonschema
  role: Used by the MESSES validate command to check the extracted JSON against the
    default experiment-description specification and user-supplied schemas
- name: mwtab
  role: Target format/library context for the messes.convert.mwtab_functions that
    transform validated JSON into mwTab JSON and tab-delimited outputs
- name: Python 3
  role: Language MESSES is implemented in and runtime requirement (Python 3.10+)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:a255585082e3557d46cea8e6a25edfd5f27c02e6cc1defd080f9e41e86fdeccb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/messes
  tools:
  - MESSES
  - Python 3
  - jsonschema
  - mwtab
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mwtab-format-conversion

## Summary

Convert tagged, validated JSONized experimental metadata and data into Metabolomics Workbench deposition-compliant mwTab outputs (both a mwTab JSON file and a mwTab tab-delimited .txt file) using the MESSES convert command with mwTab conversion directives.

## When to use

Use when a messy tabular metabolomics dataset (Excel/CSV, e.g. MS or NMR experiment metadata) must be transformed into mwTab format for Metabolomics Workbench deposition after MESSES extract and validate steps.

## When NOT to use

- The input is already in mwTab format or another repository-accepted submission format — MESSES convert targets tabular-to-JSON-to-mwTab transformation, not re-validation of finished deposits.
- The data cannot be tagged into the Experiment Description Specification schema — conversion to mwTab from extracted JSON relies on the data being in that specific schema.
- The target repository format is not mwTab — mwTab is currently the only supported conversion format in MESSES.

## Inputs

- Validated JSONized metadata/records produced by 'messes extract' from tagged tabular data (Excel or CSV)
- mwTab conversion directives / target format specification for the Metabolomics Workbench
- Optional user-supplied JSON schema (--pds) and Protocol Dependent Schema for the validate step

## Outputs

- mwTab JSON file suitable for Metabolomics Workbench deposition (e.g. output.json)
- mwTab tab-delimited file suitable for Metabolomics Workbench deposition (e.g. output.txt)

## How to apply

First tag the tabular input per the MESSES tagging system, then run 'messes extract' to turn Excel or CSV data into a JSONized intermediate representation. Run 'messes validate json your_data.json' checking against the Experiment Description Specification, the Protocol Dependent Schema (--pds), any user-supplied JSON schema, and the built-in schema specific to the target format (--format desired_format), fixing errors and repeating until clean — validate can be skipped but is not recommended. Then run 'messes convert' with the mwTab conversion directives to transform the validated JSON, which yields both a mwTab JSON file (e.g. output.json) and a mwTab tab-delimited file (e.g. output.txt), as demonstrated by the full workflow on the IC-FTMS mouse colon tissue example (Metabolomics Workbench Study ST001447). Conversion from extracted JSON relies on the data conforming to the generalized Experiment Description Specification schema, since arbitrary schemas cannot be converted. MESSES is installed via pip under Python 3.10+, and the messes console script may need to be invoked as 'python3 -m messes' on some Windows setups.

## Related tools

- **MESSES** (Python package providing the extract, validate, and convert commands used to transform tagged tabular data into mwTab deposition files) — https://github.com/MoseleyBioinformaticsLab/messes
- **jsonschema** (Used by the MESSES validate command to check the extracted JSON against the default experiment-description specification and user-supplied schemas)
- **mwtab** (Target format/library context for the messes.convert.mwtab_functions that transform validated JSON into mwTab JSON and tab-delimited outputs)
- **Python 3** (Language MESSES is implemented in and runtime requirement (Python 3.10+))

## Examples

```
messes extract your_data.csv --output your_data.json; messes validate json your_data.json --pds your_schema.json --format mwTab; messes convert mwTab your_data.json your_format_data
```

## Evaluation signals

- Both conversion artifacts exist after running convert: a mwTab JSON file (output.json) and a mwTab tab-delimited file (output.txt).
- The validated JSON passed 'messes validate' with no errors/warnings against the Experiment Description Specification, Protocol Dependent Schema, and the target-format schema before conversion.
- The output files parse as valid mwTab — JSON output is well-formed JSON and the tab-delimited output conforms to mwTab tab-delimited structure.
- Reproducing the example IC-FTMS mouse colon tissue workflow (Metabolomics Workbench Study ST001447) yields the same two output files.
- Extract-validate-fix iterations were repeated until no more errors remained, as the README workflow prescribes.

## Limitations

- Conversion from extracted JSON to another format relies on the data being in the specific Experiment Description Specification schema; arbitrary schemas cannot be converted.
- There is an initial steep learning curve for working out the extraction, validation, and conversion settings.
- The validate step can be skipped, but this is not recommended and risks emitting invalid outputs.
- Windows quirks: the messes console script may not be on PATH; use 'python3 -m messes' or the Scripts directory; Mac OS may raise SSL certificate errors requiring Install Certificates.command.

## Evidence

- [abstract] The package provides three commands, extract, validate, and convert, that implement a natural data transformation workflow.: "The package provides three commands, extract, validate, and convert, that implement a natural data transformation workflow."
- [methods] The convert command of MESSES supports converting JSON data to another JSON format or another supported format.: "The convert command of MESSES supports converting JSON data to another JSON format or another supported format."
- [readme] conversion to another format from the extracted JSON does rely on the data being in a specific schema.: [evidence span withheld — anchor status: external_doc]
- [readme] messes convert desired_format your_data.json your_format_data: [evidence span withheld — anchor status: external_doc]
- [full_text] Running the extract, validate, and convert workflow on the example IC-FTMS dataset produces two output files: a mwTab JSON file (output.json) and a mwTab tab-delimited file (output.txt), suitable for Metabolomics Workbench deposition.: [evidence span withheld — anchor status: not_found]
