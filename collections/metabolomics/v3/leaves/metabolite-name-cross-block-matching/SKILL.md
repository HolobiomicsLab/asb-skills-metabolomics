---
name: metabolite-name-cross-block-matching
description: Use when curating or validating a parsed mwTab-format study file and
  you need to confirm that metabolite names in the MS or NMR quantitative data blocks
  match the METABOLITES block, including columns labeled under variant names.
when_to_use_negative:
- The input is not an mwTab-format file (e.g., raw mzML/NetCDF spectra or an already-JSONized
  file without METABOLITES block structure)
- You need full mwTab schema validation (ms_required_schema/nmr_required_schema via
  jsonschema) rather than the metabolite-name consistency check specifically
- The file has no METABOLITES block or no MS/NMR metabolite data blocks to compare
  against
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: mwtab
  role: Provides the mwTab tokenizer, read_files generator, MWTabFile objects, and
    the NameMatcher/ValueMatcher/ColumnFinder/make_list_regex machinery that implements
    the cross-block metabolite-name consistency check inside validate_file().
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
- name: pandas
  role: Used to work with the tabular data sections of the mwTab format (the MS/NMR
    metabolite data tables) being compared by name.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:d5d4d93cd3b5d1539d5f1370bf371be95bf00ddad43333fc6b3c070cdf812d3a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/mwtab
  tools:
  - Python
  - mwtab
  - pandas
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# metabolite-name-cross-block-matching

## Summary

A quality-control check from the mwtab validator that verifies metabolite names recorded in the MS_METABOLITE_DATA / NMR_METABOLITE_DATA data blocks are consistent with the names declared in the METABOLITES block of the same mwTab file, using flexible name/value matching machinery (NameMatcher, ValueMatcher, ColumnFinder).

## When to use

Use when curating or validating a parsed mwTab-format study file and you need to confirm that metabolite names in the MS or NMR quantitative data blocks match the METABOLITES block, including columns labeled under variant names.

## When NOT to use

- The input is not an mwTab-format file (e.g., raw mzML/NetCDF spectra or an already-JSONized file without METABOLITES block structure)
- You need full mwTab schema validation (ms_required_schema/nmr_required_schema via jsonschema) rather than the metabolite-name consistency check specifically
- The file has no METABOLITES block or no MS/NMR metabolite data blocks to compare against

## Inputs

- mwTab-format study file from the Metabolomics Workbench (MS and/or NMR experimental data)
- Parsed MWTabFile object containing MS_METABOLITE_DATA, NMR_METABOLITE_DATA, and METABOLITES blocks

## Outputs

- Consistency verdict for metabolite names across the data blocks and the METABOLITES block
- Warnings/errors list (verbose output) suitable for a mwTab validation report (e.g., mwtab_validation_report.txt)

## How to apply

Parse the study file into an MWTabFile object using the mwtab tokenizer and read_files generator, then run the metabolite-name consistency sub-validation of validate_file() (Per Methods 4.5), which compares metabolite names between the MS_METABOLITE_DATA and/or NMR_METABOLITE_DATA blocks and the METABOLITES block. Because metabolite metadata columns appear under various names and formats, use the mwtab matching machinery: ColumnFinder combines a NameMatcher (whose attributes are lists of strings or lists of lists of strings applied in its dict_match method) with a ValueMatcher (whose attributes are strings applied in its series_match method) to identify columns under their various names and evaluate their value formats; regular expressions for these matchers are built with make_list_regex. Work on the tabular data sections with pandas, and run the check with verbose=True so warnings/errors for inconsistent or unmatched metabolite names are written to the validation report. The check flags cases where a metabolite name present in a data block is absent from the METABOLITES block (or vice versa), enabling repository curation and improved FAIRness.

## Related tools

- **mwtab** (Provides the mwTab tokenizer, read_files generator, MWTabFile objects, and the NameMatcher/ValueMatcher/ColumnFinder/make_list_regex machinery that implements the cross-block metabolite-name consistency check inside validate_file().) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **pandas** (Used to work with the tabular data sections of the mwTab format (the MS/NMR metabolite data tables) being compared by name.)

## Evaluation signals

- Every metabolite name appearing in MS_METABOLITE_DATA and NMR_METABOLITE_DATA blocks resolves to a name in the METABOLITES block of the same MWTabFile
- Columns of metabolite metadata are identified despite variant naming, i.e., the NameMatcher/ColumnFinder reports no unidentified metadata columns
- Running with verbose=True yields a warnings/errors listing that can be written to a validation report without uncaught exceptions
- Re-running the check on the same downloaded MW dataset produces a stable, deterministic set of inconsistencies, matching the repository-wide curation evaluation

## Limitations

- Matching depends on regular expressions built with make_list_regex; metabolite names or column headings outside the encoded NameMatcher/ValueMatcher attribute lists may be missed or misidentified
- The check is consistency-focused: it flags name mismatches between blocks but does not by itself validate numeric values or the mwTab JSON schema (those are separate sub-validations of validate_file())
- No changelog found in the source documentation, so behavioral changes between mwtab versions are hard to trace

## Evidence

- [methods] NameMatcher dict_match: "All NameMatcher attributes are lists of strings or lists of lists of strings and all are used in its only method, dict_match."
- [methods] ValueMatcher series_match: "All ValueMatcher attributes are strings and all are used in its only method, series_match."
- [methods] column identification under various names: "code was created to be able to identify the columns under their various names as well as evaluate the format of their values"
- [methods] pandas for tabular sections: "pandas_ for working with tabular data sections within the mwTab format."
- [readme] mwTab format and Metabolomics Workbench archival: "The ``mwtab`` package is a Python library that facilitates reading and writing
files in ``mwTab`` format used by the `Metabolomics Workbench`_ for archival of"
