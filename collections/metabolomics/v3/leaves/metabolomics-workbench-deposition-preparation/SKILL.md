---
name: metabolomics-workbench-deposition-preparation
description: Use when raw MS or NMR experimental metadata sits in messy spreadsheets
  (Excel/CSV) and the goal is a Metabolomics Workbench deposition in mwTab format.
when_to_use_negative:
- The output format needed is not mwTab — mwTab is currently MESSES's only supported
  conversion target, used by the Metabolomics Workbench.
- The data is untagged free-form tabular data that has not been processed with the
  MESSES tagging system and Experiment Description Specification; conversion relies
  on the extracted JSON being in the specific generalized MESSES schema.
- The goal is depositing primary raw instrument data rather than converting metadata/analysis
  tables into a repository submission format.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MESSES
  role: Python package providing the extract, validate, and convert commands that
    implement the full tagged-spreadsheet-to-mwTab transformation workflow
  repo: https://github.com/MoseleyBioinformaticsLab/messes
- name: jsonschema
  role: Schema library used by the MESSES validate command to check the extracted
    JSON against the Experiment Description Specification and user schemas
- name: mwtab
  role: Underlies the mwTab conversion functions (messes.convert.mwtab_functions)
    that emit Metabolomics Workbench-format outputs
- name: Python 3
  role: Implementation and runtime language; MESSES runs under Python 3.10+ and is
    pip-installable on Linux, Windows, and Mac
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo13070842
    title: 'MESSES: Software for Transforming Messy Research Datasets into Clean Submissions
      to Metabolomics Workbench for Public Sharing'
  source_content_hash: sha256:e52ce00ee02c3d6f5d6df003582bed69adc89b281575496ec7b8db7509ca5e8d
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
yes_rate: 0.8
n_claims: 10
---

# metabolomics-workbench-deposition-preparation

## Summary

Prepare Metabolomics Workbench-compliant deposits by running the MESSES three-command workflow — extract, validate, convert — which turns tagged tabular (Excel/CSV) metadata into JSONized records, schema-validates them with jsonschema, and emits mwTab outputs (mwTab JSON plus a tab-delimited file), as demonstrated on the IC-FTMS ST001447 example.

## When to use

Use when raw MS or NMR experimental metadata sits in messy spreadsheets (Excel/CSV) and the goal is a Metabolomics Workbench deposition in mwTab format.

## When NOT to use

- The output format needed is not mwTab — mwTab is currently MESSES's only supported conversion target, used by the Metabolomics Workbench.
- The data is untagged free-form tabular data that has not been processed with the MESSES tagging system and Experiment Description Specification; conversion relies on the extracted JSON being in the specific generalized MESSES schema.
- The goal is depositing primary raw instrument data rather than converting metadata/analysis tables into a repository submission format.

## Inputs

- Tagged tabular dataset (Excel or CSV) containing experimental metadata for an MS or NMR metabolomics study, e.g. the IC-FTMS mouse colon tissue data of Metabolomics Workbench Study ST001447
- JSON schema(s): the default Experiment Description Specification, the Protocol Dependent Schema, and optionally a user-supplied project schema (--pds)
- mwTab conversion directives selecting mwTab as the desired output format

## Outputs

- mwTab JSON file (e.g. output.json) in Metabolomics Workbench deposition format
- mwTab tab-delimited file (e.g. output.txt)
- Intermediate JSONized metadata representation (extracted records and metadata tables)

## How to apply

First add a layer of tags to the tabular data using the MESSES tagging system (read the tagging and Experiment Description Specification documentation first, since tagging must be done before extraction). Run the extract command (messes.extract.extract) to transform the tagged spreadsheet into an intermediate JSONized representation of metadata tables and records. Run the validate command (messes.validate.validate) with jsonschema against the default Experiment Description Specification, the Protocol Dependent Schema, and any user-supplied project schema, fixing errors/warnings in the original data and repeating extract-validate until clean — validate can be skipped but is not recommended. Finally run the convert command with mwTab conversion directives (messes.convert.convert / messes.convert.mwtab_functions) to produce a mwTab JSON file and a mwTab tab-delimited file suitable for Metabolomics Workbench deposition; verify both artifacts exist and match their expected formats.

## Related tools

- **MESSES** (Python package providing the extract, validate, and convert commands that implement the full tagged-spreadsheet-to-mwTab transformation workflow) — https://github.com/MoseleyBioinformaticsLab/messes
- **jsonschema** (Schema library used by the MESSES validate command to check the extracted JSON against the Experiment Description Specification and user schemas)
- **mwtab** (Underlies the mwTab conversion functions (messes.convert.mwtab_functions) that emit Metabolomics Workbench-format outputs)
- **Python 3** (Implementation and runtime language; MESSES runs under Python 3.10+ and is pip-installable on Linux, Windows, and Mac)

## Examples

```
messes extract your_data.csv --output your_data.json && messes validate json your_data.json --format mwTab && messes convert mwTab your_data.json mwTab_output
```

## Evaluation signals

- The mwTab convert step emits two artifacts: a mwTab JSON file and a mwTab tab-delimited file — confirm both exist, as in the ST001447 example (output.json and output.txt).
- messes validate json reports no errors (and ideally no warnings) against the Experiment Description Specification, the Protocol Dependent Schema, the desired-format schema, and any --pds user schema.
- Validate-checkable defects traced back to tagging errors should be fixed in the original tagged spreadsheet and the extract-validate cycle repeated until clean.
- The final files parse/inspect as valid mwTab (JSON structure and tab-delimited columns) suitable for Metabolomics Workbench deposition.
- Extraction round-trips correctly: JSONized metadata tables and records reflect all tagged spreadsheet fields for the study.

## Limitations

- There is an initial steep learning curve: users must first read the tagging and Experiment Description Specification documentation and properly tag their data before the workflow succeeds.
- Conversion to mwTab depends on the extracted JSON conforming to the specific generalized MESSES Experiment Description schema; messy manual or automated tagging can generate errors in the extracted JSON.
- The validate command can be skipped, but this is explicitly not recommended; skipping risks submitting invalid data.
- On Mac OS, SSL certificate errors ('certificate verify failed: unable to get local issuer certificate') may occur and require running Install Certificates.command; on Windows the console script may not be on the Path, requiring 'python3 -m messes' or adding the Scripts directory to Path.

## Evidence

- [abstract] The package provides three commands, extract, validate, and convert, that implement a natural data transformation workflow.: "The package provides three commands, extract, validate, and convert, that implement a natural data transformation workflow."
- [intro] MESSES (Metadata from Experimental SpreadSheets Extraction System) is a Python package that facilitates the conversion of tabular data into other formats.: "MESSES (Metadata from Experimental SpreadSheets Extraction System) is a Python package that facilitates the conversion of tabular data into other formats."
- [readme] messes extract your_data.csv --output your_data.json; messes validate json your_data.json --pds your_schema.json --format desired_format; messes convert desired_format your_data.json your_format_data: "messes extract your_data.csv --output your_data.json
   messes validate json your_data.json --pds your_schema.json --format desired_format
   messes convert desired_format your_data.json"
- [readme] Currently Supported Formats:
    
    * mwTab
        * Used by the `Metabolomics Workbench`_.: "* mwTab
        * Used by the `Metabolomics Workbench`_."
- [readme] There is an initial steep learning curve. But once the extraction, validation, and conversion settings are worked out, this process can be easily added to your data generation and analysis workflows.: "There is an initial
steep learning curve. But once the extraction, validation, and conversion settings are 
worked out, this process can be easily added to your data generation and analysis workflows."
