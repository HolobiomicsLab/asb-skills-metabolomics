---
name: restful-repository-data-retrieval
description: Use when a corpus of mwTab-format MS or NMR dataset files must be fetched
  from Metabolomics Workbench (e.g., for validator sweeps or curation audits) via
  the MW REST interface rather than manual download.
when_to_use_negative:
- Files are already available locally in mwTab format and only need parsing/validation
  — go straight to mwtab.fileio.read_files and MWTabFile.validate
- The target repository is not Metabolomics Workbench or the data are not in mwTab
  format (the library is specific to the mwTab format used by MW for MS and NMR archival)
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: mwtab
  role: Provides the MW REST interface methods and mwtab.read_files / mwtab.fileio.read_files
    used to fetch and load mwTab-format files by STUDY_ID/ANALYSIS_ID
  repo: https://github.com/MoseleyBioinformaticsLab/mwtab
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.3390/metabo11030163
    title: The mwtab Python Library for RESTful Access and Enhanced Quality Control,
      Deposition, and Curation of the Metabolomics Workbench Data Repository
  source_content_hash: sha256:330824d96bcfdb2e1364c066f12d3b7b70be67d1f1300ac9a8dad89d34ddd3e9
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
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# restful-repository-data-retrieval

## Summary

Retrieves mwTab-format study and analysis files from the Metabolomics Workbench (MW) public repository using the mwtab library's methods for interacting with the MW REST interface, as the first step of repository-wide quality-control analyses.

## When to use

Use when a corpus of mwTab-format MS or NMR dataset files must be fetched from Metabolomics Workbench (e.g., for validator sweeps or curation audits) via the MW REST interface rather than manual download.

## When NOT to use

- Files are already available locally in mwTab format and only need parsing/validation — go straight to mwtab.fileio.read_files and MWTabFile.validate
- The target repository is not Metabolomics Workbench or the data are not in mwTab format (the library is specific to the mwTab format used by MW for MS and NMR archival)

## Inputs

- Metabolomics Workbench STUDY_IDs and/or ANALYSIS_IDs identifying mwTab-format files
- MW REST endpoint accessible over the network
- mwtab Python library (v1.0.1) installed under Python 3.6+

## Outputs

- Iterable of parsed MWTabFile objects (with study_id, analysis_id, source, and mwTab blocks)
- A record of files that failed to parse (skipped, unparseable remainder of the corpus)

## How to apply

Use the mwtab Python library (v1.0.1), which exposes methods for interacting with the MW REST interface to fetch files identified by STUDY_ID or ANALYSIS_ID, as shown in the README quickstart where mwtab.read_files('1', '2') fetches files from URL by ANALYSIS_ID. Iterate over the returned mwtabfile objects, recording study_id and analysis_id for each, and feed them directly downstream into mwTab→JSON conversion or validation. For repository-wide sweeps (like the paper's tally of validation errors over 1818 parseable files), retrieve all study datasets, then use mwtab.fileio.read_files to load the files, recording and skipping any file that fails to parse so the unparseable remainder is tracked separately. Rationale: reading via the library guarantees the files conform to the mwTab format used by Metabolomics Workbench for archival of Mass Spectrometry (MS) and Nuclear Magnetic Resonance (NMR) experimental data before any validation or curation step runs.

## Related tools

- **mwtab** (Provides the MW REST interface methods and mwtab.read_files / mwtab.fileio.read_files used to fetch and load mwTab-format files by STUDY_ID/ANALYSIS_ID) — https://github.com/MoseleyBioinformaticsLab/mwtab

## Examples

```
for mwfile in mwtab.read_files("1", "2"): print("STUDY_ID:", mwfile.study_id, "ANALYSIS_ID:", mwfile.analysis_id)
```

## Evaluation signals

- Each retrieved file yields an MWTabFile whose study_id and analysis_id match the requested IDs (as printed in the README quickstart)
- Parse success rate is tracked: unparseable files are recorded and skipped, and the parseable count matches the corpus total (e.g., 1818 parseable files in the paper's sweep)
- Retrieved files expose the expected mwTab blocks (e.g., SUBJECT_SAMPLE_FACTORS, MS/NMR experimental data sections) when iterated
- Retrieval composes with downstream validation: every loaded file can be passed to MWTabFile.validate with mwschema.ms_required_schema/nmr_required_schema defaults without format errors

## Limitations

- Some repository files are unparseable by mwtab.fileio.read_files and must be recorded and skipped rather than analyzed
- The library requires Python 3.6+; installation via pip is assumed (optionally inside a virtualenv for isolation)

## Evidence

- [abstract] many new enhancements including methods for interacting with the MW REST interface: "many new enhancements including methods for interacting with the MW REST interface, enhanced format validation features, and advanced features for parsing"
- [methods] used by the Metabolomics Workbench for archival: "used by the `Metabolomics Workbench`_ for archival of Mass Spectrometry (MS) and Nuclear Magnetic Resonance (NMR) experimental data."
- [readme] library for accessing and manipulating data: "As a library for accessing and manipulating data stored in ``mwTab`` format files."
- [readme] Quickstart fetch by ANALYSIS_ID: ">>> # Here we use ANALYSIS_ID of file to fetch data from URL
>>> for mwfile in mwtab.read_files("1", "2"):"
