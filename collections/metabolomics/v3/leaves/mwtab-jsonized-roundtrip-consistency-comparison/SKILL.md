---
name: mwtab-jsonized-roundtrip-consistency-comparison
description: Use when auditing Metabolomics Workbench analyses that have both 'mwTab'
  and JSON data files, to determine which pairs are consistent and which mismatch,
  and to catalog the affected analysis IDs.
when_to_use_negative:
- The analysis has files in only one format (mwTab or JSON), so no paired comparison
  is possible
- The goal is schema-based validation of a single file rather than cross-format comparison
  — use the mwtab validate/validate_file functionality based on mwschema ms/nmr required
  schemas instead
- Input data are raw instrument files (MS or NMR) rather than already-deposited mwTab/JSON
  analysis files
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: mwtab
  role: Python library providing MWTabFile parsing (mwtab.mwtab, mwtab.fileio.read_files,
    mwtab.tokenizer) and REST access (mwtab.mwrest) to fetch both mwTab and JSONized
    files for comparison
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
- name: Python
  role: Runtime environment for the mwtab library (requires Python 3.6+)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:0b79d1d03f09937a4fe50b12f7efe404c4b6b14f57c755f6834b5f7c23c2cc35
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MoseleyBioinformaticsLab/mwtab
  tools:
  - Python
  - mwtab
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mwtab-jsonized-roundtrip-consistency-comparison

## Summary

A QC workflow for the Metabolomics Workbench repository that retrieves paired mwTab and JSONized files for analyses available in both formats, parses them into MWTabFile objects, and systematically compares section keys, item keys/values, SUBJECT_SAMPLE_FACTORS, and _DATA sections to quantify format-inconsistency.

## When to use

Use when auditing Metabolomics Workbench analyses that have both 'mwTab' and JSON data files, to determine which pairs are consistent and which mismatch, and to catalog the affected analysis IDs.

## When NOT to use

- The analysis has files in only one format (mwTab or JSON), so no paired comparison is possible
- The goal is schema-based validation of a single file rather than cross-format comparison — use the mwtab validate/validate_file functionality based on mwschema ms/nmr required schemas instead
- Input data are raw instrument files (MS or NMR) rather than already-deposited mwTab/JSON analysis files

## Inputs

- mwTab-format analysis files from the Metabolomics Workbench REST interface
- Equivalent JSONized-format analysis files from the Metabolomics Workbench REST interface
- List of ANALYSIS_IDs with files present in both formats (e.g., 1655 paired analyses)

## Outputs

- Per-category mismatch tallies (total mismatched analyses, section-key mismatches, item key/value mismatches, SUBJECT_SAMPLE_FACTORS mismatches, _DATA section mismatches)
- List of analysis IDs for every mismatched pair
- CSV consistency report (e.g., mwtab_format_consistency_report.csv)

## How to apply

Retrieve both format variants via the Metabolomics Workbench REST interface (mwtab.mwrest) for each analysis with files present in both formats. Parse every file into an MWTabFile instance using mwtab.mwtab with mwtab.fileio.read_files and mwtab.tokenizer, which converts between the mwTab format and its equivalent JSONized representation. For each paired parsed object, compare in order: section (block) keys, item keys and item values, SUBJECT_SAMPLE_FACTORS entries, and _DATA section contents. Tally mismatches per category (e.g., section-key mismatches vs. item-key/value vs. SUBJECT_SAMPLE_FACTORS vs. _DATA, such as JSON files missing a 'Data' item in AN001492, AN001493, AN001499, AN002428). Write tallies and the list of mismatched analysis IDs (e.g., AN001492, AN001493, AN001499, AN002428, AN000441, AN001960, AN002398, AN002403, AN002404) to a CSV report such as mwtab_format_consistency_report.csv.

## Related tools

- **mwtab** (Python library providing MWTabFile parsing (mwtab.mwtab, mwtab.fileio.read_files, mwtab.tokenizer) and REST access (mwtab.mwrest) to fetch both mwTab and JSONized files for comparison) — https://github.com/MoseleyBioinformaticsLab/mwtab
- **Python** (Runtime environment for the mwtab library (requires Python 3.6+))

## Examples

```
import mwtab; for mwfile in mwtab.read_files("AN001492", "AN001493"): print(mwfile.analysis_id, list(mwfile.keys()))
```

## Evaluation signals

- Reproduced total of mismatched analyses matches the reported figure (1345 mismatched out of 1685 compared analyses)
- All named affected analysis IDs (AN001492, AN001493, AN001499, AN002428, AN000441, AN001960, AN002398, AN002403, AN002404) appear in the output report
- Subtype mismatch counts (section keys, item keys/values, SUBJECT_SAMPLE_FACTORS, _DATA sections) sum consistently with the per-analysis tallies
- Every compared pair parses into an MWTabFile without tokenizer/fileio errors, and both members of each pair have identical analysis_id

## Limitations

- The comparison covers only analyses for which data files exist in both mwTab and JSON formats; analyses with a single format are out of scope
- Mismatches discovered include missing items (e.g., JSON files missing a 'Data' item), so round-trip conversion via the JSONized representation is not guaranteed lossless for all analyses
- REST retrieval of ~1655 paired files can be slow and subject to network/repository availability

## Evidence

- [abstract] many new enhancements including methods for interacting with the MW REST interface, enhanced format validation features, and advanced features for parsing and searching for specific metabolite data: "many new enhancements including methods for interacting with the MW REST interface, enhanced format validation features, and advanced features for parsing and searching for specific metabolite data"
- [methods] convert mwTab files into JSON representation and vice versa: "convert ``mwTab`` formatted files into their equivalent JSONized (JavaScript Object Notation, an open-standard format that uses human-readable text to transmit data objects"
- [readme] README: JSONized conversion facilities: [evidence span withheld — anchor status: external_doc]
