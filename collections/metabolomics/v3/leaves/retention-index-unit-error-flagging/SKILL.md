---
name: retention-index-unit-error-flagging
description: Use when auditing METABOLITES section headers and value series in mwTab
  files to detect retention-index columns labeled with time units (e.g. minutes) or
  other logical-format mismatches against standardized fields like pubchem_id, inchi_key,
  hmdb_id.
when_to_use_negative:
- Input is already JSONized mwTab data or a validated/deposited file whose column
  naming has been curated — only raw mwTab METABOLITES sections need unit-error flagging
- Analysis goal is format/schema validation of the whole mwTab file via mwschema MS/NMR
  required schemas rather than semantic column-name and value-series checks
- Study does not use mwTab format (e.g. RAW mzML or other formats not processed by
  the mwtab package)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: mwtab
  role: Parses mwTab files, provides MWTabFile objects whose METABOLITES blocks are
    searched with ColumnFinder/NameMatcher/ValueMatcher regular expressions to flag
    unit errors
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
- name: pandas
  role: Builds and writes the summary CSV of per-column match counts, per-file flags,
    and recorded logical errors from the tabular data sections
- name: Python
  role: Runtime environment (3.6+) in which the mwtab library and the regular-expression
    matching workflow are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:752b971323bfa9e8a077f4cff656b758c8e83bed406383258d55a3e8e039270a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/mwtab
  tools:
  - Python
  - mwtab
  - pandas
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# retention-index-unit-error-flagging

## Summary

Flags logical unit errors in metabolite metadata columns — e.g. 'retention index (min)' — by combining mwtab's ColumnFinder/NameMatcher regular-expression matching of standardized field names with ValueMatcher evaluation of value-series formats across Metabolomics Workbench mwTab files.

## When to use

Use when auditing METABOLITES section headers and value series in mwTab files to detect retention-index columns labeled with time units (e.g. minutes) or other logical-format mismatches against standardized fields like pubchem_id, inchi_key, hmdb_id.

## When NOT to use

- Input is already JSONized mwTab data or a validated/deposited file whose column naming has been curated — only raw mwTab METABOLITES sections need unit-error flagging
- Analysis goal is format/schema validation of the whole mwTab file via mwschema MS/NMR required schemas rather than semantic column-name and value-series checks
- Study does not use mwTab format (e.g. RAW mzML or other formats not processed by the mwtab package)

## Inputs

- mwTab format files from the Metabolomics Workbench repository (MS or NMR analyses)
- standard_column_names.json containing standardized metadata column names
- column_finders dictionary of ColumnFinder objects with NameMatcher and ValueMatcher regular expressions

## Outputs

- Per-column match counts for standardized metabolite metadata fields
- Per-file 'has matching common fields' flags
- Flagged logical errors such as 'retention index (min)' unit annotations
- Well-formed summary CSV listing all standardized columns

## How to apply

Parse each deposited mwTab file with the mwtab package (read_files / MWTabFile) and extract the METABOLITES section column headers and value series. Load the standardized column names from standard_column_names.json and instantiate the column_finders dictionary of ColumnFinder objects built from modular regular expressions assembled with make_list_regex. For each standardized column (retention_index, retention_time, pubchem_id, inchi_key, kegg_id, hmdb_id, etc.) run dict_match to match column names under aliases (e.g. 'm/z' vs 'moverz') and series_match to evaluate the value formats, such as HMDB IDs with or without a lower/upper-case 'HMDB' prefix. When a column name matched as 'retention index' carries time-unit annotations like 'retention index (min)', record it as a flagged logical error, since retention index values are unitless. Aggregate per-column match counts and per-file 'has matching common fields' flags into a summary CSV for curation review.

## Related tools

- **mwtab** (Parses mwTab files, provides MWTabFile objects whose METABOLITES blocks are searched with ColumnFinder/NameMatcher/ValueMatcher regular expressions to flag unit errors) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **pandas** (Builds and writes the summary CSV of per-column match counts, per-file flags, and recorded logical errors from the tabular data sections)
- **Python** (Runtime environment (3.6+) in which the mwtab library and the regular-expression matching workflow are executed)

## Examples

```
for mwfile in mwtab.read_files('1818'):
    flags = [col for col in mwfile['METABOLITES'] if dict_match(col, column_finders) and series_match(col, 'retention index (min)')]
```

## Evaluation signals

- Summary CSV exists, is well-formed tabular output, and lists all standardized columns from the column_finders dictionary
- Reproduced file-level tally matches the reported 1216 of 1818 files containing matching common fields
- Every flagged logical error corresponds to a matched retention-index column annotated with a time unit (e.g. 'retention index (min)')
- Series_match results for ID fields agree with expected formats, e.g. HMDB IDs match both with and without a lower/upper-case 'HMDB' prefix
- Aggregated per-column match counts cover each standardized field including retention_index and retention_time

## Limitations

- Column-name matching depends on alias lists; variants outside the assembled regular expressions (e.g. unusual spellings of 'moverz' for 'm/z') will be missed
- Value-series format evaluation is heuristic; IDs or unit annotations deviating from anticipated patterns may not be flagged
- Results reflect the corpus of 1818 deposited mwTab files evaluated for curation and FAIRness of the Metabolomics Workbench repository; newer or non-MW datasets are not covered

## Evidence

- [methods] metadata column matching of metabolite metadata columns: "code was created to be able to identify the columns under their various names as well as evaluate the format of their values"
- [methods] unit error record: "All NameMatcher attributes are lists of strings or lists of lists of strings and all are used in its only method, dict_match."
- [methods] value format evaluation: "All ValueMatcher attributes are strings and all are used in its only method, series_match."
- [methods] regex assembly: "There is also a function used heavily in creating these regular expressions, make_list_regex."
- [abstract] validation goal: "We used the enhanced format validation features to evaluate all available datasets in MW to facilitate improved curation and FAIRness of the repository."
- [readme] parsing entry point: [evidence span withheld — anchor status: external_doc]
