---
name: subject-sample-factors-metadata-verification
description: Use when Metabolomics Workbench analyses are available in both 'mwTab'
  and JSON formats and you must verify whether SUBJECT_SAMPLE_FACTORS entries (and
  related section/item content) agree between the two parsed representations.
when_to_use_negative:
- Only one format is available for an analysis — no paired mwTab/JSON comparison is
  possible.
- The goal is schema validation of a single mwTab file rather than cross-format consistency;
  use mwtab's validate/validate_file with mwschema ms/nmr required schemas instead.
- Input is a non-Metabolomics-Workbench metabolomics format (e.g., mzML or NetCDF);
  the mwTab MWTabFile parser does not apply.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: mwtab
  role: Python library used to fetch files via the MW REST interface, parse mwTab
    and JSONized files into MWTabFile objects, and convert between formats for comparison
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
- name: Python
  role: Runtime for the comparison workflow (mwtab runs under Python 3.6+)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:3f9f3fb81d5f138e9596403cc3549ab2747b66f445ec082377dd7b849ca7b29e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/mwtab
  tools:
  - Python
  - mwtab
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# subject-sample-factors-metadata-verification

## Summary

Compares the SUBJECT_SAMPLE_FACTORS metadata sections of paired Metabolomics Workbench files to detect mismatches between the mwTab text representation and its JSONized counterpart, as part of cross-format consistency checking of metabolomics analyses.

## When to use

Use when Metabolomics Workbench analyses are available in both 'mwTab' and JSON formats and you must verify whether SUBJECT_SAMPLE_FACTORS entries (and related section/item content) agree between the two parsed representations.

## When NOT to use

- Only one format is available for an analysis — no paired mwTab/JSON comparison is possible.
- The goal is schema validation of a single mwTab file rather than cross-format consistency; use mwtab's validate/validate_file with mwschema ms/nmr required schemas instead.
- Input is a non-Metabolomics-Workbench metabolomics format (e.g., mzML or NetCDF); the mwTab MWTabFile parser does not apply.

## Inputs

- mwTab-format analysis files from Metabolomics Workbench (retrieved via mwtab.mwrest)
- JSONized-format analysis files for the same ANALYSIS_IDs
- List of ANALYSIS_IDs available in both formats (e.g., 1655 analyses)

## Outputs

- Per-category mismatch tallies (section keys, item keys/values, SUBJECT_SAMPLE_FACTORS, _DATA sections)
- List of mismatched analysis IDs (e.g., AN001492, AN001493, AN001499, AN002428)
- Consistency report CSV (e.g., mwtab_format_consistency_report.csv)

## How to apply

Retrieve both the mwTab-format and JSONized files for each analysis via the Metabolomics Workbench REST interface (mwtab.mwrest). Parse every file into an MWTabFile instance using mwtab.mwtab with mwtab.fileio.read_files and mwtab.tokenizer so both formats are held in a common object model. For each paired parsed object, first compare section keys, then item keys and values, then the SUBJECT_SAMPLE_FACTORS entries, and finally the _DATA section data; tally mismatches per category (e.g., 1345 mismatched analyses out of 1685 compared, 14 with differing section keys). Record the analysis IDs of every mismatched pair (e.g., AN001492, AN001493, AN001499, AN002428, which had JSON files missing a 'Data' item) and write tallies plus affected IDs to a report such as mwtab_format_consistency_report.csv. The rationale is that the mwTab package mirrors both archival formats, so differences between parsed representations reveal curation or conversion defects in the repository.

## Related tools

- **mwtab** (Python library used to fetch files via the MW REST interface, parse mwTab and JSONized files into MWTabFile objects, and convert between formats for comparison) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **Python** (Runtime for the comparison workflow (mwtab runs under Python 3.6+))

## Examples

```
import mwtab; mwfiles = list(mwtab.read_files("AN001492", "AN001493"))
```

## Evaluation signals

- Reproduced mismatch tally matches the reported figure of 1345 inconsistent analyses out of 1685 compared (1655 analyses available in both formats per retrieval step).
- Section-key mismatch count matches the reported 14 analyses with different section keys.
- All named affected analysis IDs (AN001492, AN001493, AN001499, AN002428, AN000441, AN001960, AN002398, AN002403, AN002404) appear in the output report.
- AN002428 and similar analyses show the expected defect: the JSON file is missing a 'Data' item.
- The report CSV (mwtab_format_consistency_report.csv) contains both the per-category tallies and the affected analysis ID list.

## Limitations

- The large majority of paired analyses were inconsistent between formats, so absence of a match is the expected outcome for most pairs, not evidence of a parsing error.
- Comparison results depend on the mwtab package version staying synchronized with repository format changes (mwtab 1.0.1 at time of publication).
- Coverage is limited to the 1655 analyses with data files present in both formats; analyses lacking one representation cannot be checked.

## Evidence

- [other] REST retrieval and parsing workflow: [evidence span withheld — anchor status: not_found]
- [other] parsing with MWTabFile, fileio, tokenizer: [evidence span withheld — anchor status: not_found]
- [methods] format conversion rationale: "convert ``mwTab`` formatted files into their equivalent JSONized (JavaScript Object Notation, an open-standard format that uses human-readable text to transmit data objects"
- [methods] tokenizer module: ".. automodule:: mwtab.tokenizer"
- [methods] read_files function: ".. automodule:: mwtab.fileio

.. autofunction:: read_files"
- [readme] README quickstart usage: ">>> for mwfile in mwtab.read_files("1", "2"):
...      print("STUDY_ID:", mwfile.study_id)"
