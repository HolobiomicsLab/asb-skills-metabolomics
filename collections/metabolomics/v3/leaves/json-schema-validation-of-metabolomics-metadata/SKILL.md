---
name: json-schema-validation-of-metabolomics-metadata
description: Use when tabular experimental metadata (Excel/CSV) has been extracted
  to JSON via MESSES and must be checked against the Experiment Description Specification,
  a PDS, or custom JSON schema before conversion to mwTab.
when_to_use_negative:
- Data has not yet been extracted by `messes extract` — there is no JSON to validate;
  extraction must come first in the workflow
- The goal is conversion to a format other than MESSES-supported formats (currently
  mwTab is the only supported output), where the built-in format schema check does
  not apply
- Data already conforms to Metabolomics Workbench mwTab and needs no MESSES-side schema
  checking
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MESSES
  role: Provides the `validate` command that checks extracted JSON against the Experiment
    Description Specification, a user-supplied Protocol Dependent Schema, and built-in
    format-specific schemas
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: jsonschema
  role: Python library backing MESSES' JSON schema validation of the extracted metadata
- name: mwtab
  role: Related tooling for the mwTab format that the validated JSON is ultimately
    converted into
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:be20680f55b80e2f948deedd76c09a2d68e7b925a34d2cdac742f184f7ed54a9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/messes
  tools:
  - MESSES
  - jsonschema
  - mwtab
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# json-schema-validation-of-metabolomics-metadata

## Summary

Validates MESSES-extracted JSONized metadata against the Experiment Description Specification, a Protocol Dependent Schema (PDS), user-supplied JSON schemas, and a format-specific schema before converting to a deposition format such as mwTab.

## When to use

Use when tabular experimental metadata (Excel/CSV) has been extracted to JSON via MESSES and must be checked against the Experiment Description Specification, a PDS, or custom JSON schema before conversion to mwTab.

## When NOT to use

- Data has not yet been extracted by `messes extract` — there is no JSON to validate; extraction must come first in the workflow
- The goal is conversion to a format other than MESSES-supported formats (currently mwTab is the only supported output), where the built-in format schema check does not apply
- Data already conforms to Metabolomics Workbench mwTab and needs no MESSES-side schema checking

## Inputs

- JSONized metadata produced by `messes extract` from tagged Excel or CSV tabular data
- Protocol Dependent Schema (PDS) as a user-supplied JSON schema file (e.g. your_schema.json)
- Target conversion format identifier (e.g. mwTab)

## Outputs

- Validation report of errors and warnings against the Experiment Description Specification, PDS, user schemas, and format-specific schema
- Schema-valid JSON ready for `messes convert` into mwTab JSON / mwTab tab-delimited outputs (output.json, output.txt)

## How to apply

After running `messes extract` on the tagged spreadsheet, run the `messes validate json` command on the emitted JSON, passing the Protocol Dependent Schema via `--pds` (a user-supplied JSON schema) and the target format via `--format` so that MESSES applies its built-in format-specific schema in addition to the default Experiment Description Specification checks; validation itself is backed by the jsonschema Python library. Fix every reported error and warning in the original tabular data, re-extract, and repeat until the run is error-free — only then proceed to `messes convert`. Validation may be skipped, but MESSES documentation explicitly recommends against skipping it because manual tagging of spreadsheets commonly introduces errors into the extracted JSONized representation.

## Related tools

- **MESSES** (Provides the `validate` command that checks extracted JSON against the Experiment Description Specification, a user-supplied Protocol Dependent Schema, and built-in format-specific schemas) — https://github.com/MoseleyBioinformaticsLab/messes
- **jsonschema** (Python library backing MESSES' JSON schema validation of the extracted metadata)
- **mwtab** (Related tooling for the mwTab format that the validated JSON is ultimately converted into)

## Examples

```
messes validate json your_data.json --pds your_schema.json --format mwTab
```

## Evaluation signals

- `messes validate json` completes with no errors and no warnings against the Experiment Description Specification and the supplied PDS
- Extracted JSON conforms to the Experiment Description Specification (checked with jsonschema) and to the schema for the chosen format (e.g. mwTab)
- Iteration terminates: the documented loop of extract → validate → fix errors/warnings in original data → repeat ends with an error-free run before conversion
- Downstream conversion succeeds and yields both expected artifacts, e.g. mwTab JSON (output.json) and mwTab tab-delimited file (output.txt) for the ST001447 IC-FTMS example

## Limitations

- Validation is schema-based only: it cannot detect semantically wrong but schema-valid metadata entered during manual tagging of spreadsheets
- Conversion to another format requires the data to be in MESSES' specific Experiment Description schema; arbitrary JSON schemas usable for extraction will not convert
- There is an initial steep learning curve to MESSES tagging and schema setup before the extract/validate/convert workflow runs cleanly
- MESSES runs under Python 3.10+, and older pip versions are known to have issues grabbing dependencies

## Evidence

- [abstract] The package provides three commands, extract, validate, and convert, that implement a natural data transformation workflow.: "The package provides three commands, extract, validate, and convert, that implement a natural data transformation workflow."
- [methods] The validate command of MESSES supports validating JSON data.: "The validate command of MESSES supports validating JSON data."
- [readme] The validation step ensures the data that was extracted is valid against the Experiment Description Specification: [evidence span withheld — anchor status: external_doc]
- [readme] use the "validate" command to validate the JSON based on your specific project schema, fix errors and warnings: "then use the "validate" command to validate the JSON based on your specific
project schema, fix errors and warnings in the original data, repeat steps 1-3 until
there are no more errors"
- [readme] messes validate json your_data.json --pds your_schema.json --format desired_format: [evidence span withheld — anchor status: external_doc]
