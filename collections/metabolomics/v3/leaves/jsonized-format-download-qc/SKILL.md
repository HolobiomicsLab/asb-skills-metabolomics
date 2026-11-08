---
name: jsonized-format-download-qc
description: Use when auditing the download integrity and parseability of a large
  batch of Metabolomics Workbench analyses (e.g. all 1891 analyses) in the JSONized
  format, to identify blank/un-downloadable and un-parseable entries for curation.
when_to_use_negative:
- Input is a single already-downloaded mwTab or JSON file that just needs conversion
  or validation — use mwtab's convert/validate functionality instead of a repository-wide
  download audit
- The data source is not the Metabolomics Workbench REST interface (this procedure
  depends on MW analysis IDs and the mwrest module)
- Only the mwTab format is of interest — the mwTab-format tallies (3 un-downloadable,
  70 un-parseable) differ from the JSONized tallies and require their own download
  pass
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: mwtab
  role: Provides mwtab.mwrest for RESTful retrieval of analyses and mwtab.fileio.read_files
    with the mwtab tokenizer to parse downloaded JSONized files into MWTabFile objects
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
- name: pandas
  role: Working with tabular sections within the mwTab format and aggregating the
    download/parse QC counts
- name: Python
  role: Runtime environment for the download-and-parse audit script (mwtab runs under
    Python 3.6+)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:9f1d759046756daffa78bed404aee460571e6e956fbd17d9cbadffaa00796072
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
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# jsonized-format-download-qc

## Summary

A repository-wide quality-control procedure that enumerates every Metabolomics Workbench analysis through the mwTab REST interface, downloads each in JSONized format, and tallies entries that return blank pages or fail to parse into MWTabFile objects via the mwtab tokenizer.

## When to use

Use when auditing the download integrity and parseability of a large batch of Metabolomics Workbench analyses (e.g. all 1891 analyses) in the JSONized format, to identify blank/un-downloadable and un-parseable entries for curation.

## When NOT to use

- Input is a single already-downloaded mwTab or JSON file that just needs conversion or validation — use mwtab's convert/validate functionality instead of a repository-wide download audit
- The data source is not the Metabolomics Workbench REST interface (this procedure depends on MW analysis IDs and the mwrest module)
- Only the mwTab format is of interest — the mwTab-format tallies (3 un-downloadable, 70 un-parseable) differ from the JSONized tallies and require their own download pass

## Inputs

- Full list of Metabolomics Workbench analysis IDs obtainable via the MW REST interface (e.g. 1891 analyses in the 19 Nov 2020 snapshot)
- mwtab Python package (v1.0.1) with mwrest, fileio.read_files, and tokenizer modules
- mwTab and JSONized data files served by the Metabolomics Workbench REST interface

## Outputs

- Per-format tally of un-downloadable analyses (blank-page entries) for the JSONized format
- Per-format tally of files that raise parse errors when read into MWTabFile objects
- Aggregated download/parse quality-control counts table (e.g. 50 JSON un-downloadable, 139 JSON un-parseable, 53 blank entries)

## How to apply

Enumerate the full set of available analyses through the Metabolomics Workbench REST interface using the mwtab.mwrest module (the 19 November 2020 snapshot contained 1891 analyses). Download every analysis in JSONized format, recording entries that fail to download or return only blank pages, and keep these counts separate from parse failures. For each successfully downloaded file, attempt to parse it into MWTabFile objects using mwtab.fileio.read_files with the mwtab tokenizer, tallying files that raise parse errors. In the reference run, 50 of 1891 analyses could not be downloaded in JSON format (with only blank pages present) and 139 downloaded JSON files were un-parseable, alongside 53 blank entries overall. Aggregate the tallies into a counts table mirroring the paper's download/parse quality-control summary, comparing JSON results against the mwTab-format results (3 un-downloadable, 70 un-parseable) to expose format-specific curation gaps.

## Related tools

- **mwtab** (Provides mwtab.mwrest for RESTful retrieval of analyses and mwtab.fileio.read_files with the mwtab tokenizer to parse downloaded JSONized files into MWTabFile objects) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **pandas** (Working with tabular sections within the mwTab format and aggregating the download/parse QC counts)
- **Python** (Runtime environment for the download-and-parse audit script (mwtab runs under Python 3.6+))

## Examples

```
import mwtab
for mwfile in mwtab.read_files("1", "2"):
    print("ANALYSIS_ID:", mwfile.analysis_id)
```

## Evaluation signals

- Total enumerated analyses matches the REST interface count (1891 in the 19 Nov 2020 snapshot) — every analysis is attempted in JSONized format
- Un-downloadable count equals the number of entries returning only blank pages (50 for JSONized in the reference run), kept distinct from parse failures
- Parse-failure tally (139 for JSONized) is reproducible by re-running mwtab.fileio.read_files and counting raised exceptions per format
- The aggregated counts table reconciles: attempted = downloadable + un-downloadable, with the mwTab-format tallies (3, 70) reported alongside for cross-format comparison
- Blank entries (53 across formats) are counted once per file and match the difference between failed downloads and empty responses

## Limitations

- Failed downloads manifest as blank pages rather than HTTP errors, so the audit must inspect file content (blank-page detection), not just request status
- Counts are snapshot-specific: results reflect the 19 November 2020 state of the Metabolomics Workbench and will shift as the repository is curated and updated
- JSONized and mwTab formats fail at different rates (50 vs 3 un-downloadable; 139 vs 70 un-parseable), so a single-format audit understates repository-wide quality issues
- The mwtab package version matters — behavior mirrors the mwTab and JSONized formats as of the released version (1.0.1); no changelog was found in the article's discussion

## Evidence

- [results] 50 of 1891 analyses could not be downloaded in JSON format, with only blank pages present: [evidence span withheld — anchor status: not_found]
- [abstract] mwtab provides methods for interacting with the MW REST interface: "many new enhancements including methods for interacting with the MW REST interface, enhanced format validation features"
- [readme] read_files parses downloads into MWTabFile objects with the tokenizer: [evidence span withheld — anchor status: external_doc]
- [methods] convert mwTab files into JSON representation and vice versa: "convert ``mwTab`` formatted files into their equivalent JSONized (JavaScript Object Notation, an open-standard format that uses human-readable text to transmit data objects"
