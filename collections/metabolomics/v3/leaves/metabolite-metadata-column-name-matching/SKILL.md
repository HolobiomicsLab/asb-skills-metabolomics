---
name: metabolite-metadata-column-name-matching
description: Use when analyzing mwTab-format metabolomics files whose METABOLITES
  section headers must be mapped to standardized field names and value formats for
  repository curation, FAIRness assessment, or quality control.
when_to_use_negative:
- When the input is not an mwTab-formatted file (e.g. raw mzML/NetCDF spectra or a
  finished feature table) — the matching operates on METABOLITES section headers and
  value series
- When only a single-column lookup is needed without format validation of the values
  — NameMatcher/ValueMatcher machinery is designed for whole-column curation sweeps
- When files were deposited after the paper's 1818-file snapshot and have not been
  checked against the alias/value regex lists
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: mwtab
  role: Python library used to read mwTab files and expose METABOLITES section headers
    and value series for column matching
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
- name: pandas
  role: Used for working with tabular data sections within the mwTab format and assembling
    summary counts
- name: make_list_regex
  role: Helper function heavily used to build the modular regular expressions behind
    NameMatcher and ValueMatcher pattern lists
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:90d94ac9fdd28a54270c2b6f04a79f0ee2a42cb92800c8c52f2ab2ee0cd92f14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/mwtab
  tools:
  - Python
  - make_list_regex
  - mwtab
  - pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# metabolite-metadata-column-name-matching

## Summary

Identify metabolite metadata columns in mwTab files under their many aliases (e.g. pubchem_id, inchi_key, hmdb_id, retention time/index) using regular-expression NameMatchers and validate their value formats with ValueMatchers, enabling automated curation and reproducibility checks on the Metabolomics Workbench repository.

## When to use

Use when analyzing mwTab-format metabolomics files whose METABOLITES section headers must be mapped to standardized field names and value formats for repository curation, FAIRness assessment, or quality control.

## When NOT to use

- When the input is not an mwTab-formatted file (e.g. raw mzML/NetCDF spectra or a finished feature table) — the matching operates on METABOLITES section headers and value series
- When only a single-column lookup is needed without format validation of the values — NameMatcher/ValueMatcher machinery is designed for whole-column curation sweeps
- When files were deposited after the paper's 1818-file snapshot and have not been checked against the alias/value regex lists

## Inputs

- mwTab-format files from Metabolomics Workbench (MS or NMR experimental data)
- standard_column_names.json mapping standardized fields to alias and value regex lists

## Outputs

- Per-standardized-column match counts across the file corpus
- Per-file flag 'has matching common fields' (e.g. 1216 of 1818 files)
- Flagged logical errors such as 'retention index (min)' unit mismatches
- Summary CSV listing all standardized columns from the column_finders dictionary

## How to apply

Parse each mwTab file with the mwtab package (mwtab.read_files / MWTabFile) and extract the METABOLITES section column headers and their value series. Build a column_finders dictionary of ColumnFinder objects, each carrying a NameMatcher and a ValueMatcher whose pattern lists are assembled with the make_list_regex helper from standard_column_names.json. Use dict_match to match column names against aliases (e.g. 'm/z' versus 'moverz', retention_time variants) and series_match to verify value formats such as HMDB IDs with or without a lower/upper-case 'HMDB' prefix. Aggregate per-column match counts and per-file 'has matching common fields' flags, and flag logical errors like 'retention index (min)' unit mislabeling. The paper used this to determine that 1216 of 1818 downloaded mwTab files contained headers matching the standardized common fields.

## Related tools

- **mwtab** (Python library used to read mwTab files and expose METABOLITES section headers and value series for column matching) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **pandas** (Used for working with tabular data sections within the mwTab format and assembling summary counts)
- **make_list_regex** (Helper function heavily used to build the modular regular expressions behind NameMatcher and ValueMatcher pattern lists) — https://github.com/MoseleyBioinformaticsLab/mwtab

## Examples

```
for mwfile in mwtab.read_files("1", "2"):
    cols = mwfile["METABOLITES"]["sample"]
    matches = {c: dict_match(col_finders[c], cols) for c in col_finders}
```

## Evaluation signals

- Summary CSV exists, is well-formed tabular output, and lists all standardized columns present in the column_finders dictionary
- Reproduced file-level tally of 'has matching common fields' equals the reported 1216 of 1818 files
- Per-column counts are consistent with the alias mappings (e.g. 'm/z' matched to the 'moverz' alias)
- Value-format validation catches malformed identifiers, e.g. HMDB IDs missing a lower/upper-case 'HMDB' prefix
- Logical-error flags such as 'retention index (min)' unit mislabeling are surfaced in the output

## Limitations

- Matching depends on the alias and value regex lists in standard_column_names.json; novel column names or value formats absent from that dictionary will be missed
- The evaluation covered the 1818 mwTab files available at the time; newly deposited or non-mwTab datasets require re-running the sweep
- Column-name matching alone does not confirm correct data semantics — value-format and logical-error checks (e.g. 'retention index (min)') are needed, and even these cannot catch all curation issues

## Evidence

- [methods] metadata column matching of metabolite metadata columns: "code was created to be able to identify the columns under their various names as well as evaluate the format of their values"
- [methods] All NameMatcher attributes are lists of strings or lists of lists of strings and all are used in its only method, dict_match: "All NameMatcher attributes are lists of strings or lists of lists of strings and all are used in its only method, dict_match"
- [methods] All ValueMatcher attributes are strings and all are used in its only method, series_match: "All ValueMatcher attributes are strings and all are used in its only method, series_match"
- [methods] There is also a function used heavily in creating these regular expressions, make_list_regex: "There is also a function used heavily in creating these regular expressions, make_list_regex"
- [readme] The mwtab package is a Python library that facilitates reading and writing files in mwTab format used by the Metabolomics Workbench: "The ``mwtab`` package is a Python library that facilitates reading and writing
files in ``mwTab`` format used by the `Metabolomics Workbench`_"
