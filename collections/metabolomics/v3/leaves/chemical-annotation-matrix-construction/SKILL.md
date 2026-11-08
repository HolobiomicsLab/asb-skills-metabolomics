---
name: chemical-annotation-matrix-construction
description: Use when the goal is a repository-scale, file-level chemical annotation
  table (e.g., compound presence/absence across thousands of public MS/MS files) built
  from ReDU/GNPS library search output rather than from raw spectra.
when_to_use_negative:
- You need de novo or structural annotation beyond level 2/3 spectral reference matching
  — GNPS library annotations are only level 2 (spectral library similarity) or level
  3 by the 2007 metabolomics standards initiative.
- You need molecular networking relationships (edges between spectra) rather than
  per-file annotation presence/absence — use GNPS molecular networking (recommended
  file limit 3000) instead.
- Your data is not in MassIVE / not yet indexed by ReDU — ReDU only interacts with
  MassIVE and updates periodically, so recently contributed data will not display
  immediately.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: ReDU
  role: Serves the compoundslist of all annotated chemicals and provides the File
    Selector / Chemical Explorer used to validate the matrix
  repo: https://github.com/mwang87/ReDU-MS2-GNPS
- name: GNPS
  role: Hosts the pre-computed MS/MS spectral library search jobs (batched at 5000
    files/job) whose results populate the matrix
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
- name: MassIVE
  role: Public repository holding the underlying files and the ReDU identification
    database (MSV000084206); ReDU only interacts with MassIVE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-020-0916-7
    title: 'ReDU: a framework to find and reanalyze public mass spectrometry data'
  source_content_hash: sha256:29250fde8cd89a94266bb284b5a9da8e8e7c640f6f2926355c1617976c89b96c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/mwang87/ReDU-MS2-Documentation
  repo_url: https://github.com/mwang87/ReDU-MS2-GNPS
  tools:
  - GNPS
  - MassIVE
  - ReDU
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# chemical-annotation-matrix-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces ReDU's repository-wide chemical annotation table by retrieving pre-computed GNPS spectral library search results (MS/MS spectral library matching) for all public MassIVE files and encoding them as a binary chemical-annotation-by-file matrix.

## When to use

Use when the goal is a repository-scale, file-level chemical annotation table (e.g., compound presence/absence across thousands of public MS/MS files) built from ReDU/GNPS library search output rather than from raw spectra.

## When NOT to use

- You need de novo or structural annotation beyond level 2/3 spectral reference matching — GNPS library annotations are only level 2 (spectral library similarity) or level 3 by the 2007 metabolomics standards initiative.
- You need molecular networking relationships (edges between spectra) rather than per-file annotation presence/absence — use GNPS molecular networking (recommended file limit 3000) instead.
- Your data is not in MassIVE / not yet indexed by ReDU — ReDU only interacts with MassIVE and updates periodically, so recently contributed data will not display immediately.

## Inputs

- ReDU compoundslist (all compounds annotated across the ReDU corpus, from https://redu.ucsd.edu/compoundslist)
- Pre-computed GNPS spectral library search results for public MassIVE files (ReDU identification database MSV000084206; batched task tsv e.g. database/global_tasks.tsv)
- ReDU batch template dump from https://redu.ucsd.edu/metabatchdump

## Outputs

- gnps_library_search_annotations.tsv — binary annotation-by-file matrix, cells coded 1 (detected) / 0 (not detected), rows = compoundslist compounds, columns = public ReDU files

## How to apply

Download the published chemical annotation list served by the ReDU Chemical Explorer from https://redu.ucsd.edu/compoundslist, which enumerates all compounds annotated across the ReDU corpus. Retrieve the GNPS spectral library search results that ReDU has pre-computed for all public MassIVE files (ReDU's identification database is GNPS/MassIVE accession MSV000084206), using default library search parameters and respecting the GNPS recommended limit of 5000 files per library search job (for larger collections, jobs are batched via the ReDU metabatchdump template and the Batch Workflow for Spectral Library Search, with task IDs collected as tsv, e.g. database/global_tasks.tsv). Parse the results into file-level annotations, retaining the multiple GNPS annotations a single chemical can receive because slight variation in MS2 spectra (m/z or abundance) matches different reference spectra. Encode each cell as 1 if the compound was annotated in that file and 0 otherwise, and write gnps_library_search_annotations.tsv with rows matching compoundslist entries and columns matching ReDU public files.

## Related tools

- **ReDU** (Serves the compoundslist of all annotated chemicals and provides the File Selector / Chemical Explorer used to validate the matrix) — https://github.com/mwang87/ReDU-MS2-GNPS
- **GNPS** (Hosts the pre-computed MS/MS spectral library search jobs (batched at 5000 files/job) whose results populate the matrix) — https://github.com/CCMS-UCSD/GNPSDocumentation
- **MassIVE** (Public repository holding the underlying files and the ReDU identification database (MSV000084206); ReDU only interacts with MassIVE)

## Examples

```
curl -O https://redu.ucsd.edu/compoundslist && python build_annotation_matrix.py --compounds compoundslist --tasks database/global_tasks.tsv --out gnps_library_search_annotations.tsv
```

## Evaluation signals

- Matrix rows correspond 1:1 to entries in the compoundslist downloaded from https://redu.ucsd.edu/compoundslist.
- Matrix columns correspond to the public files reported by ReDU; spot-check per-file annotation counts against the 'All Data in ReDU' view of the ReDU Chemical Explorer.
- Spot-check compound identities in the matrix against the ReDU Chemical Explorer (e.g. 'Spectral Match to Sulfachloropyridazine from NIST14' style annotation names).
- Cells are strictly binary (1/0): 1 where the chemical was annotated in the file, 0 otherwise.
- Multiple GNPS annotations for the same chemical (from slight MS2 m/z or abundance variation) are all retained rather than collapsed.

## Limitations

- GNPS recommended limit of 5000 files per library search job, so repository-wide reproduction requires batched jobs collected as task tsvs.
- A single chemical can receive multiple GNPS annotations due to slight variation in MS2 spectra (m/z or abundance) matching different reference spectra.
- Library annotations are only level 2 or level 3 confidence under the 2007 metabolomics standards initiative.
- ReDU only interacts with MassIVE and data are updated periodically; recently contributed data are not displayed immediately.

## Evidence

- [results] The library search results are displayed on GNPS and can be downloaded as a table of every chemical annotation versus the files it was detected in, with matrix cells coded 1 (detected) or 0 (not detected).: [evidence span withheld — anchor status: not_found]
- [abstract] Recommended file limits: "The recommended file limits are 3000 for molecular networking in GNPS and 5000 for library search in GNPS"
- [abstract] Multiple annotations per chemical: "Slight variation in the MS2 spectra (m/z or abundance) cause the pattern to match different reference MS2 spectra"
- [abstract] Identification database accession: "The ReDU identification database is publicly available and accessible via (GNPS), MSV000084206"
- [readme] Batched library search update procedure: "Download batch template for GNPS at ```https://redu.ucsd.edu/metabatchdump```
1. Run Batch Workflow for Spectral Library Search
1. Get the set of tasks as tsv and save to"
- [methods] ReDU repository scope: "ReDU only interacts with MassIVE; however, there are plans to incorporate data and metadata from other resources."
