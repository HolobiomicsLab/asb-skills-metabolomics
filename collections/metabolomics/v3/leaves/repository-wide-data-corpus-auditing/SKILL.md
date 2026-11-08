---
name: repository-wide-data-corpus-auditing
description: Use when you need to quality-control a whole repository's holdings rather
  than one file — e.g. counting how many Metabolomics Workbench analyses fail to download
  or fail mwTab/JSON parsing at a given snapshot date.
when_to_use_negative:
- You only need to read or convert a handful of known analysis IDs — use mwtab's normal
  read_files/conversion path instead of a corpus-wide audit.
- The target repository exposes neither a REST enumeration endpoint nor bulk download
  in multiple formats — the sweep design depends on both.
- Your goal is content-level validation of metabolite metadata columns (e.g. NameMatcher/ValueMatcher
  dict_match/series_match checks) rather than downloadability and parseability of
  the whole corpus.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3071
tools:
- name: mwtab
  role: Python library providing mwrest REST access, fileio.read_files parsing with
    the mwtab tokenizer, and mwTab<->JSON conversion used to download and parse the
    full corpus
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
- name: pandas
  role: Aggregating the download/parse failure tallies into a tabular counts summary
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:f4454f3ecdbd9744d7a1887720a0c8cb73216e76969aef0587811165f7a99982
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
  - http://edamontology.org/topic_3071
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# repository-wide-data-corpus-auditing

## Summary

Systematically enumerate and download an entire data-repository corpus (all 1891 Metabolomics Workbench analyses) in multiple formats (mwTab and JSONized) via the REST interface, then attempt to parse every file to quantify download and parse failures for repository curation and FAIRness assessment.

## When to use

Use when you need to quality-control a whole repository's holdings rather than one file — e.g. counting how many Metabolomics Workbench analyses fail to download or fail mwTab/JSON parsing at a given snapshot date.

## When NOT to use

- You only need to read or convert a handful of known analysis IDs — use mwtab's normal read_files/conversion path instead of a corpus-wide audit.
- The target repository exposes neither a REST enumeration endpoint nor bulk download in multiple formats — the sweep design depends on both.
- Your goal is content-level validation of metabolite metadata columns (e.g. NameMatcher/ValueMatcher dict_match/series_match checks) rather than downloadability and parseability of the whole corpus.

## Inputs

- List of analysis IDs from the Metabolomics Workbench REST interface (full enumeration of available analyses)
- Metabolomics Workbench REST endpoints for mwTab and JSONized downloads

## Outputs

- Per-format download-failure tally (entries that fail to download or return blank pages)
- Per-format parse-failure tally (files raising errors in mwtab.fileio.read_files)
- Aggregated counts table mirroring the paper's download/parse QC summary (3 mwTab + 50 JSON un-downloadable; 70 mwTab + 139 JSON un-parseable)

## How to apply

First enumerate the full corpus through the repository REST API using mwtab.mwrest (1891 analyses available as of the 19 November 2020 snapshot). Download every analysis twice — once in native mwTab format and once in the JSONized representation — recording separately any entries that fail to download or return blank files. Then attempt to parse each downloaded file into MWTabFile objects with mwtab.fileio.read_files using the mwtab tokenizer, tallying files that raise parse errors per format. Aggregate the tallies into a counts table mirroring the paper's quality-control summary: 3 mwTab + 50 JSON un-downloadable (53 blank entries) and 70 mwTab + 139 JSON un-parseable. The dual-format audit is the rationale: discrepancies between mwTab and JSON failure counts expose format-conversion and curation defects in the repository itself.

## Related tools

- **mwtab** (Python library providing mwrest REST access, fileio.read_files parsing with the mwtab tokenizer, and mwTab<->JSON conversion used to download and parse the full corpus) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **pandas** (Aggregating the download/parse failure tallies into a tabular counts summary)

## Examples

```
import mwtab; files = list(mwtab.read_files(*analysis_ids))  # parse each downloaded mwTab analysis, catching tokenizer errors
```

## Evaluation signals

- Corpus enumeration matches the repository's advertised analysis count (1891 analyses at the 19 November 2020 snapshot).
- Blank/failed downloads are recorded per format and the two formats' tallies are reported separately, not merged.
- Parse failures are counted only among files that downloaded successfully, distinguishing un-downloadable from un-parseable entries.
- Final counts reproduce the paper's QC summary: 3 mwTab + 50 JSON un-downloadable (53 blank entries) and 70 mwTab + 139 JSON un-parseable.
- Every downloaded file was actually attempted through mwtab.fileio.read_files (parser exceptions captured, not swallowed silently).

## Limitations

- Results are snapshot-dependent: counts reflect the 19 November 2020 state of the Metabolomics Workbench and will drift as entries are curated.
- 'Could not be downloaded' included files that returned only blank pages, so failure classification depends on how blank responses are detected.
- The audit measures format validity/parseability, not scientific correctness of the underlying data.

## Evidence

- [other] Query the Metabolomics Workbench through the mwtab.mwrest module to enumerate the full set of 1891 available analyses.: [evidence span withheld — anchor status: not_found]
- [abstract] We used the enhanced format validation features to evaluate all available datasets in MW to facilitate improved curation and FAIRness of the repository.: "We used the enhanced format validation features to evaluate all available datasets in MW to facilitate improved curation and FAIRness of the repository."
- [methods] read_files is the parsing entry point used to convert downloaded files into MWTabFile objects.: ".. automodule:: mwtab.fileio

.. autofunction:: read_files"
- [readme] README quickstart demonstrates reading multiple analyses by ID via read_files.: ">>> # Here we use ANALYSIS_ID of file to fetch data from URL
   >>> for mwfile in mwtab.read_files("1", "2"):"
