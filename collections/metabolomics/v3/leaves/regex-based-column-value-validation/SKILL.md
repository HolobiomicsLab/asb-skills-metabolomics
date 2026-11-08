---
name: regex-based-column-value-validation
description: Use when parsing mwTab METABOLITES sections and needing to detect standardized
  fields (e.g. pubchem_id, inchi_key, kegg_id, hmdb_id, retention_time) regardless
  of column-name variants or value formats.
when_to_use_negative:
- Input is not an mwTab file (e.g. raw MS/NMR instrument output or already-JSONized
  data) — the column/value matchers are built for the mwTab METABOLITES section layout
- The goal is validating required MS/NMR schema fields via mwTab.validate rather than
  matching metabolite metadata columns by alias and value format
- Columns are already fully standardized so alias matching and value-format validation
  would add no information
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: mwtab
  role: Parses mwTab files into Python objects (read_files / MWTabFile) whose METABOLITES
    column headers and value series feed the NameMatcher and ValueMatcher regular
    expressions
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
- name: pandas
  role: Holds the extracted tabular data sections and supports assembling the summary
    CSV of per-column match counts
- name: jsonschema
  role: Complementary validation of mwTab file structure against MS/NMR required schemas;
    used alongside the column/value matchers for quality control
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:41256b84f3b1426dcefdd1c1233c4f18f6da27560a4ffeec598ac1ca3ae8557a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/mwtab
  tools:
  - Python
  - jsonschema
  - mwtab
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# regex-based-column-value-validation

## Summary

A method for identifying metabolite metadata columns in mwTab files under their many aliases and validating the format of their values using modular regular expressions, used to standardize and flag logical errors in Metabolomics Workbench deposits.

## When to use

Use when parsing mwTab METABOLITES sections and needing to detect standardized fields (e.g. pubchem_id, inchi_key, kegg_id, hmdb_id, retention_time) regardless of column-name variants or value formats.

## When NOT to use

- Input is not an mwTab file (e.g. raw MS/NMR instrument output or already-JSONized data) — the column/value matchers are built for the mwTab METABOLITES section layout
- The goal is validating required MS/NMR schema fields via mwTab.validate rather than matching metabolite metadata columns by alias and value format
- Columns are already fully standardized so alias matching and value-format validation would add no information

## Inputs

- mwTab files from the Metabolomics Workbench repository (full library of 1818 datasets)
- standard_column_names.json mapping standardized column names to aliases and expected value formats

## Outputs

- Summary CSV listing per-column match counts for every standardized column in the column_finders dictionary
- Per-file 'has matching common fields' flag
- Flagged logical errors such as 'retention index (min)' units

## How to apply

Parse each mwTab file with the mwtab package (read_files / MWTabFile) and extract the METABOLITES section column headers and their value series. Load the standardized column names from standard_column_names.json and build a column_finders dictionary of ColumnFinder objects, each pairing a NameMatcher with a ValueMatcher whose attributes are lists of strings generated with make_list_regex so that aliases like 'm/z' vs 'moverz' are matched and value formats (e.g. HMDB IDs with or without a lower/upper-case 'HMDB' prefix) are recognized. For each standardized column, run NameMatcher.dict_match against the column headers and ValueMatcher.series_match against the value series; column names and values that match a standardized field are aggregated. Flag logical inconsistencies such as 'retention index (min)' units, and tally per-column match counts plus a per-file 'has matching common fields' flag. Validate by confirming the summary CSV is well-formed, lists every standardized column from the column_finders dictionary, and reproduces the article's count of 1216 of 1818 files with matching common fields.

## Related tools

- **mwtab** (Parses mwTab files into Python objects (read_files / MWTabFile) whose METABOLITES column headers and value series feed the NameMatcher and ValueMatcher regular expressions) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **pandas** (Holds the extracted tabular data sections and supports assembling the summary CSV of per-column match counts)
- **jsonschema** (Complementary validation of mwTab file structure against MS/NMR required schemas; used alongside the column/value matchers for quality control)

## Examples

```
import mwtab; mwfiles = list(mwtab.read_files("1","2")); for m in mwfiles: print(m.analysis_id, list(m.keys()))
```

## Evaluation signals

- Reproduced per-file tally matches the article's reported 1216 of 1818 files containing metabolite metadata headings matching standardized fields
- Summary CSV is well-formed tabular output and lists all standardized columns from the column_finders dictionary
- Logical errors such as 'retention index (min)' units are flagged in the summary
- Aliases like 'm/z' vs 'moverz' and HMDB IDs with or without a lower/upper-case 'HMDB' prefix are correctly matched by dict_match and series_match

## Limitations

- Counts depend on the alias lists in standard_column_names.json and the regular expressions assembled with make_list_regex; unmatched variants of a standardized field are silently missed
- Only the 1818 deposited mwTab files were evaluated; files lacking a METABOLITES section or with non-standard metadata headings fall outside the matched set

## Evidence

- [methods] metadata column matching of metabolite metadata columns: "code was created to be able to identify the columns under their various names as well as evaluate the format of their values"
- [methods] regular expression assembly for column matchers: "There is also a function used heavily in creating these regular expressions, make_list_regex."
- [methods] NameMatcher dict_match mechanism: "All NameMatcher attributes are lists of strings or lists of lists of strings and all are used in its only method, dict_match."
- [methods] ValueMatcher series_match mechanism: "All ValueMatcher attributes are strings and all are used in its only method, series_match."
- [methods] ColumnFinder pairing of matchers: "a ColumnFinder has a NameMatcher and ValueMatcher to handle those functions."
- [results] task finding: 1216 of 1818 files matched: [evidence span withheld — anchor status: not_found]
- [readme] mwTab parsing with mwtab package: "The ``mwtab`` package is a Python library that facilitates reading and writing files in ``mwTab`` format used by the `Metabolomics Workbench`_"
