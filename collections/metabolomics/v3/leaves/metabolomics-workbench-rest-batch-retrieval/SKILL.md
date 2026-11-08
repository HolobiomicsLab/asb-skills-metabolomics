---
name: metabolomics-workbench-rest-batch-retrieval
description: Use when you need a complete, current inventory of Metabolomics Workbench
  analyses retrieved via REST (e.g. 1891 analyses at the 19 Nov 2020 snapshot) and
  per-format download/parse failure counts for curation or FAIRness assessment.
when_to_use_negative:
- You only need a single known analysis file — a direct REST fetch of that ANALYSIS_ID
  suffices and a repository-wide census is wasteful.
- The data source is not the Metabolomics Workbench repository (e.g. MetaboLights
  or SRA) — the mwTab/JSONized formats and mwrest module are MW-specific.
- The goal is reformatting or schema validation of files you already hold locally
  rather than assessing retrievability across the repository.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: mwtab
  role: Python library providing mwrest for REST access, fileio.read_files with the
    mwtab tokenizer for parsing, and mwTab/JSON conversion; used to enumerate, download,
    and parse all MW analyses
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
- name: Python
  role: Runtime (Python 3.6+) in which the mwtab package batch-retrieval and parsing
    loop is executed
- name: pandas
  role: Working with tabular data sections within the mwTab format and aggregating
    the download/parse failure counts
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:0f160b68896ce07b18141a4e7fe6660c06f1306c8eb152df2e120186e3548f10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/mwtab
  tools:
  - Python
  - mwtab
  - pandas
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metabolomics-workbench-rest-batch-retrieval

## Summary

Enumerate all analyses available through the Metabolomics Workbench REST interface and batch-download each in both mwTab and JSONized formats, tallying un-downloadable (blank) files and files that fail parsing into MWTabFile objects, as a repository-wide quality-control census.

## When to use

Use when you need a complete, current inventory of Metabolomics Workbench analyses retrieved via REST (e.g. 1891 analyses at the 19 Nov 2020 snapshot) and per-format download/parse failure counts for curation or FAIRness assessment.

## When NOT to use

- You only need a single known analysis file — a direct REST fetch of that ANALYSIS_ID suffices and a repository-wide census is wasteful.
- The data source is not the Metabolomics Workbench repository (e.g. MetaboLights or SRA) — the mwTab/JSONized formats and mwrest module are MW-specific.
- The goal is reformatting or schema validation of files you already hold locally rather than assessing retrievability across the repository.

## Inputs

- List of Metabolomics Workbench analysis IDs (enumerated via mwtab.mwrest)
- Metabolomics Workbench REST interface endpoints for mwTab and JSONized file retrieval

## Outputs

- Per-format download success/failure tallies (e.g. 3 mwTab, 50 JSON un-downloadable; 53 blank entries)
- Per-format parse-error tallies (e.g. 70 mwTab, 139 JSON un-parseable)
- Aggregated counts table summarizing repository-wide download and parse quality control

## How to apply

Query the MW REST interface through the mwtab.mwrest module to enumerate the full set of available analyses (1891 in the 19 November 2020 snapshot). Download every analysis in mwTab format via the REST interface, recording entries that fail to download or return blank files, then repeat the batch in the JSONized format, tallying un-downloadable entries separately. Attempt to parse each downloaded file into MWTabFile objects using mwtab.fileio.read_files with the mwtab tokenizer, counting files that raise parse errors per format. Aggregate the tallies into a counts table mirroring the paper's QC summary — at the reference snapshot: 3 mwTab and 50 JSON un-downloadable, 53 blank entries, and 70 mwTab plus 139 JSON un-parseable.

## Related tools

- **mwtab** (Python library providing mwrest for REST access, fileio.read_files with the mwtab tokenizer for parsing, and mwTab/JSON conversion; used to enumerate, download, and parse all MW analyses) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **Python** (Runtime (Python 3.6+) in which the mwtab package batch-retrieval and parsing loop is executed)
- **pandas** (Working with tabular data sections within the mwTab format and aggregating the download/parse failure counts)

## Examples

```
import mwtab; analyses = mwtab.mwrest.get_all_analysis_ids();
for mwfile in mwtab.read_files("1", "2"): print("ANALYSIS_ID:", mwfile.analysis_id)
```

## Evaluation signals

- The enumerated analysis count matches the expected repository size (1891 analyses at the 19 November 2020 snapshot).
- Blank downloads are detected: files that return empty pages are counted as un-downloadable rather than as successful retrievals (53 blank entries expected at the reference snapshot).
- mwTab and JSON failure counts are tallied independently; JSON failures (50 un-downloadable, 139 un-parseable) should exceed mwTab failures (3 un-downloadable, 70 un-parseable) as in the paper.
- Every successfully downloaded file parses into an MWTabFile object via mwtab.fileio.read_files without raising, confirming the tokenizer handled the format.
- The final counts table reproduces the paper's download/parse quality-control summary structure (downloaded, un-downloadable, blank, parse errors per format).

## Limitations

- Some analyses return blank pages at the REST endpoints for one or both formats, so failure to retrieve does not imply the analysis is absent from the repository.
- Failure counts are snapshot-specific: the 3 mwTab / 50 JSON un-downloadable and 70 mwTab / 139 JSON un-parseable tallies reflect the 19 November 2020 state and will drift as the repository is curated.
- The package runs under Python 3.6+ and its formats must mirror ongoing changes in the mwTab and JSONized representations; older library versions may fail on newer depositions.

## Evidence

- [supplementary] Query the Metabolomics Workbench through the mwtab.mwrest module to enumerate the full set of 1891 available analyses.: [evidence span withheld — anchor status: not_found]
- [supplementary] 3 could not be downloaded in 'mwTab' format and 50 could not be downloaded in JSON format: [evidence span withheld — anchor status: not_found]
- [abstract] many new enhancements including methods for interacting with the MW REST interface, enhanced format validation features: "many new enhancements including methods for interacting with the MW REST interface, enhanced format validation features, and advanced features for parsing and searching for specific metabolite data"
- [readme] The ``mwtab`` package is a Python library that facilitates reading and writing files in ``mwTab`` format: "The ``mwtab`` package is a Python library that facilitates reading and writing
files in ``mwTab`` format used by the `Metabolomics Workbench`_ for archival of"
- [abstract] We used the enhanced format validation features to evaluate all available datasets in MW: "We used the enhanced format validation features to evaluate all available datasets in MW to facilitate improved curation and FAIRness of the repository."
