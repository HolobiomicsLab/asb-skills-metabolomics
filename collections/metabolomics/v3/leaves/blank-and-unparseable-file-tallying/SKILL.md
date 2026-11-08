---
name: blank-and-unparseable-file-tallying
description: 'Use when auditing the integrity of a file-based public data repository:
  you need to enumerate all records via a REST interface, fetch each in one or more
  serialized formats, and report how many return blank files or raise parser errors.'
when_to_use_negative:
- Input is a single local mwTab or JSON file you simply want to convert or validate
  — use mwtab's convert/validate functions instead of a repository-wide tally
- The repository endpoint returns structured error codes rather than blank pages,
  so failures can be counted from HTTP responses without attempting to parse each
  file
- You only need schema validation of known-good records (validate against mwschema.ms_required_schema
  / nmr_schema) rather than a census of un-downloadable and un-parseable files
tools:
- name: mwtab
  role: Provides the mwrest module to enumerate and download analyses, and the fileio.read_files
    function with the mwtab tokenizer to attempt parsing of each downloaded file
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
- name: pandas
  role: Aggregating the per-format download and parse tallies into a counts table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:cebecf53378d290f1a6a5a493d9d812d21a57e440acc83adb60fa2b5d949e4e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/mwtab
  tools:
  - Python
  - mwtab
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# blank-and-unparseable-file-tallying

## Summary

A repository-wide quality-control tally that downloads every available Metabolomics Workbench analysis in both mwTab and JSONized formats and counts files that download blank or fail to parse into MWTabFile objects, mirroring the download/parse QC summary of the mwtab 1.0.1 paper.

## When to use

Use when auditing the integrity of a file-based public data repository: you need to enumerate all records via a REST interface, fetch each in one or more serialized formats, and report how many return blank files or raise parser errors.

## When NOT to use

- Input is a single local mwTab or JSON file you simply want to convert or validate — use mwtab's convert/validate functions instead of a repository-wide tally
- The repository endpoint returns structured error codes rather than blank pages, so failures can be counted from HTTP responses without attempting to parse each file
- You only need schema validation of known-good records (validate against mwschema.ms_required_schema / nmr_schema) rather than a census of un-downloadable and un-parseable files

## Inputs

- REST enumeration of all available Metabolomics Workbench analysis IDs (1891 analyses as of the 19 November 2020 snapshot)
- Downloaded files in mwTab format
- Downloaded files in JSONized (JSON) representation

## Outputs

- Per-format counts of un-downloadable (blank) files (e.g. 3 mwTab, 50 JSON)
- Per-format counts of un-parseable files (e.g. 70 mwTab, 139 JSON)
- Aggregated download/parse quality-control summary table (e.g. built with pandas)

## How to apply

Enumerate the full set of available records first (the paper enumerated 1891 analyses through the MW REST interface via mwtab.mwrest). Download every record in each target format separately — here mwTab and JSONized — and record entries that fail to download or return blank pages as un-downloadable per format. Then attempt to parse each successfully downloaded file into MWTabFile objects using mwtab.fileio.read_files with the mwtab tokenizer, tallying files that raise parse errors for each format. Aggregate into a per-format counts table: the paper found 3 mwTab and 50 JSON un-downloadable (53 blank entries total) and 70 mwTab and 139 JSON un-parseable. The rationale is that format-conversion bugs and blank depositions surface only when every record is exercised, so per-format failure categories must be kept separate to make curation actionable.

## Related tools

- **mwtab** (Provides the mwrest module to enumerate and download analyses, and the fileio.read_files function with the mwtab tokenizer to attempt parsing of each downloaded file) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **pandas** (Aggregating the per-format download and parse tallies into a counts table)

## Examples

```
for mwfile in mwtab.read_files("1", "2"):
    print("ANALYSIS_ID:", mwfile.analysis_id)
```

## Evaluation signals

- The enumeration step returns the expected total record count (1891 analyses in the paper's snapshot); a different total indicates an incomplete REST query
- Un-downloadable + un-parseable counts are reported separately per format; blank-page entries are counted in the un-downloadable category, not mixed into parse errors
- Every successfully downloaded file yields a MWTabFile object or a recorded parse error — no silently skipped files
- The final counts table matches the paper's shape: mwTab un-downloadable=3, JSON un-downloadable=50, blank entries=53, mwTab un-parseable=70, JSON un-parseable=139
- Re-running the tally on the same snapshot reproduces identical counts (deterministic, snapshot-stamped)

## Limitations

- Results are snapshot-dependent: the paper's counts apply to analyses available as of 19 November 2020; later depositions change the tallies
- Failures are classified only as 'blank page' (un-downloadable) or 'parse error' (un-parseable); other failure modes are not distinguished
- The tally detects broken files but does not by itself fix or re-curate them — it flags entries for downstream curation

## Evidence

- [abstract] many new enhancements including methods for interacting with the MW REST interface, enhanced format validation features, and advanced features for parsing and searching for specific metabolite data: "many new enhancements including methods for interacting with the MW REST interface, enhanced format validation features, and advanced features for parsing"
- [abstract] We used the enhanced format validation features to evaluate all available datasets in MW to facilitate improved curation and FAIRness of the repository.: "We used the enhanced format validation features to evaluate all available datasets in MW to facilitate improved curation and FAIRness"
- [methods] .. automodule:: mwtab.fileio

.. autofunction:: read_files: ".. automodule:: mwtab.fileio

.. autofunction:: read_files"
- [readme] The ``mwtab`` package is a Python library that facilitates reading and writing files in ``mwTab`` format used by the `Metabolomics Workbench`_: "The ``mwtab`` package is a Python library that facilitates reading and writing files in ``mwTab`` format used by the"
