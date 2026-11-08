---
name: gnps-library-search-results-retrieval
description: Use when the goal is a repository-scale chemical annotation table across
  public tandem MS files, e.g. reproducing the ReDU compound-by-file GNPS library
  search matrix or updating ReDU's identification database.
when_to_use_negative:
- The goal is molecular networking (molecular similarity networks) rather than library-search
  annotations — GNPS molecular networking has a different recommended file limit of
  3000 files.
- Only files within a user-selected metadata-filtered cohort (G1–G6 via the ReDU File
  Selector) are needed, rather than all public files in ReDU.
- Data are not in MassIVE — ReDU only interacts with MassIVE and does not yet incorporate
  other resources.
tools:
- name: ReDU
  role: Source of the compoundslist, batch template, Chemical Explorer validation
    view, and the pre-computed library search results over all public MassIVE files
  repo: https://github.com/mwang87/ReDU-MS2-GNPS
- name: GNPS
  role: Platform whose spectral library search (MS/MS spectral library matching) produces
    the chemical annotations retrieved here; account and workflow required
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
- name: MassIVE
  role: Public repository holding the files searched and hosting the ReDU identification
    database (MSV000084206)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-020-0916-7
    title: 'ReDU: a framework to find and reanalyze public mass spectrometry data'
  source_content_hash: sha256:1d7b5b655ba1e75969b6522e013662ad5435081f63f3d90ae6f321b7a142505d
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# gnps-library-search-results-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieve ReDU's pre-computed GNPS spectral library search results (MS/MS spectral library matching with default parameters) for all public MassIVE files and encode them as a binary chemical-annotation-by-file matrix with 1 = detected and 0 = not detected.

## When to use

Use when the goal is a repository-scale chemical annotation table across public tandem MS files, e.g. reproducing the ReDU compound-by-file GNPS library search matrix or updating ReDU's identification database.

## When NOT to use

- The goal is molecular networking (molecular similarity networks) rather than library-search annotations — GNPS molecular networking has a different recommended file limit of 3000 files.
- Only files within a user-selected metadata-filtered cohort (G1–G6 via the ReDU File Selector) are needed, rather than all public files in ReDU.
- Data are not in MassIVE — ReDU only interacts with MassIVE and does not yet incorporate other resources.

## Inputs

- ReDU compoundslist from https://redu.ucsd.edu/compoundslist (all compounds annotated across the ReDU corpus)
- ReDU identification database on GNPS/MassIVE (accession MSV000084206)
- Pre-computed GNPS MS/MS spectral library search results for public MassIVE files
- ReDU batch template from https://redu.ucsd.edu/metabatchdump (when re-running the Batch Workflow for Spectral Library Search)

## Outputs

- gnps_library_search_annotations.tsv — binary chemical-annotation-by-file matrix (1 = detected, 0 = not detected)
- File-level annotation lists retaining multiple GNPS annotations per chemical

## How to apply

First download the published compound enumeration served by the ReDU Chemical Explorer at https://redu.ucsd.edu/compoundslist and locate the ReDU identification database on GNPS/MassIVE (accession MSV000084206). Then retrieve the GNPS spectral library search results that ReDU pre-computed over all public MassIVE files using default library search parameters, respecting the GNPS recommended limit of 5000 files per library search job (batch template available at https://redu.ucsd.edu/metabatchdump via the Batch Workflow for Spectral Library Search). When parsing, retain the multiple GNPS annotations a single chemical can receive because slight variation in MS2 spectra (m/z or abundance) matches different reference MS2 spectra, so rows correspond to annotation entries rather than unique compounds only. Finally, build the binary annotation-by-file matrix (1 = annotation present in a file, 0 = absent) and write it as a TSV, e.g. gnps_library_search_annotations.tsv.

## Related tools

- **ReDU** (Source of the compoundslist, batch template, Chemical Explorer validation view, and the pre-computed library search results over all public MassIVE files) — https://github.com/mwang87/ReDU-MS2-GNPS
- **GNPS** (Platform whose spectral library search (MS/MS spectral library matching) produces the chemical annotations retrieved here; account and workflow required) — https://github.com/CCMS-UCSD/GNPSDocumentation
- **MassIVE** (Public repository holding the files searched and hosting the ReDU identification database (MSV000084206))

## Examples

```
curl -L https://redu.ucsd.edu/compoundslist -o compoundslist.tsv
```

## Evaluation signals

- Spot-check per-file annotation counts in the matrix against the 'All Data in ReDU' view of the ReDU Chemical Explorer.
- Matrix rows must correspond to entries in the compoundslist downloaded from https://redu.ucsd.edu/compoundslist.
- Matrix columns must correspond to the public files reported by ReDU/MassIVE.
- Matrix cells must be strictly binary (1 or 0).
- Multiple GNPS annotations for the same chemical (due to MS2 m/z or abundance variation) must be retained as separate rows rather than collapsed.

## Limitations

- GNPS library search is capped at a recommended 5000 files per job, so repository-scale searches are chunked.
- GNPS spectral library annotations are only level 2 or level 3 by the 2007 metabolomics standards initiative (putative annotations, not definitive identifications).
- ReDU data are updated periodically and will not display recently contributed data immediately, so retrieved results may lag the repository.
- A single chemical can carry multiple GNPS annotations because slight MS2 variation matches different reference spectra.

## Evidence
