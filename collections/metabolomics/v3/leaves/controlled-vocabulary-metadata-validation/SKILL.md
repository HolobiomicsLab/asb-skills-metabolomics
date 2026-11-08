---
name: controlled-vocabulary-metadata-validation
description: 'Use when preparing or checking a ReDU sample-information table (gnps_metadata.tsv
  schema: Filename, MassIVE accession, SampleType, ...) before File Selector cohort
  assembly or contribution to ReDU.'
when_to_use_negative:
- Input is a set of raw or converted MS/MS files (mzXML/mzML) with no metadata table
  — this skill validates metadata, not spectra
- The analysis goal is chemical annotation itself (GNPS spectral library search or
  molecular networking) rather than the metadata grouping/validation step that precedes
  it
- Data originate from repositories other than MassIVE (e.g. MetaboLights or Metabolomics
  Workbench), which ReDU does not currently ingest directly
edam_topics:
- http://edamontology.org/topic_0219
tools:
- name: ReDU
  role: Provides the metadata template, validator, controlled vocabularies, Attribute
    Filters Panel, and File Selector used to validate metadata and assemble G1–G6
    cohorts
  repo: https://github.com/mwang87/ReDU-MS2-GNPS
- name: MassIVE
  role: Repository against which validated metadata rows must match; validator unit
    tests confirm the same number of files are found in MassIVE
- name: ReDU-MS2-Documentation
  role: Documentation site with instructions for contributing data and sample information
    (HowtoContribute)
  repo: https://github.com/mwang87/ReDU-MS2-Documentation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-020-0916-7
    title: 'ReDU: a framework to find and reanalyze public mass spectrometry data'
  source_content_hash: sha256:3d4548eafaf27122f40232ed1a9ba2578ee8e8529a5ad4e056264c0dbe9517c5
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
  - MassIVE
  - ReDU
  - ReDU-MS2-Documentation
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# controlled-vocabulary-metadata-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates ReDU sample-information (metadata) tables against the controlled vocabularies and MassIVE file-matching rules that ReDU enforces before files can be selected, grouped (G1–G6), and launched for GNPS analysis.

## When to use

Use when preparing or checking a ReDU sample-information table (gnps_metadata.tsv schema: Filename, MassIVE accession, SampleType, ...) before File Selector cohort assembly or contribution to ReDU.

## When NOT to use

- Input is a set of raw or converted MS/MS files (mzXML/mzML) with no metadata table — this skill validates metadata, not spectra
- The analysis goal is chemical annotation itself (GNPS spectral library search or molecular networking) rather than the metadata grouping/validation step that precedes it
- Data originate from repositories other than MassIVE (e.g. MetaboLights or Metabolomics Workbench), which ReDU does not currently ingest directly

## Inputs

- ReDU sample information (metadata) table in gnps_metadata.tsv schema (Filename, MassIVE accession, SampleType, ...)
- ReDU batch template downloaded from https://redu.ucsd.edu/metabatchdump
- MassIVE dataset identifiers (e.g. MSV000084206) associated with the files

## Outputs

- Validated sample-information table passing ReDU controlled-vocabulary validation
- Tab-separated manifest mapping each selected Filename to its group (G1–G6) plus applied metadata filters

## How to apply

Complete the ReDU sample information template (manual completion per the methods workflow), then run it through the ReDU validators, which distinguish valid files (passing validation and matching the same number of files in MassIVE), invalid files, and files that pass validation but fail to match MassIVE data. Fields must use ReDU's validated controlled vocabularies; the 'ML import: not available' abbreviation marks rows whose data were imported from MetaboLights rather than MassIVE, since ReDU only interacts with MassIVE. For cohort assembly, apply the Attribute Filters Panel semantics — active filters appear as red-boxed attributes — to select files whose metadata match chosen controlled-vocabulary values, then assign selected filenames to groups G1–G6. Enforce the grouping constraints: library search uses only Group 1 (G1), molecular networking accepts at most 6 groups, and recommended file limits are 3000 files for molecular networking and 5000 for library search. Emit the final manifest as a tab-separated table mapping each Filename to its group and the applied metadata filters.

## Related tools

- **ReDU** (Provides the metadata template, validator, controlled vocabularies, Attribute Filters Panel, and File Selector used to validate metadata and assemble G1–G6 cohorts) — https://github.com/mwang87/ReDU-MS2-GNPS
- **MassIVE** (Repository against which validated metadata rows must match; validator unit tests confirm the same number of files are found in MassIVE)
- **ReDU-MS2-Documentation** (Documentation site with instructions for contributing data and sample information (HowtoContribute)) — https://github.com/mwang87/ReDU-MS2-Documentation

## Examples

```
make test-push
```

## Evaluation signals

- Validated files match the same number of files in MassIVE (the valid_* unit-test naming convention)
- Emitted manifest is a tab-separated table with Filename and group columns and no more than 6 distinct groups
- No group other than G1 is assigned to the library-search role
- Every active metadata filter appears as a red box in the Attribute Filters Panel (upper-right corner of the page)
- Selected file counts stay within recommended limits: 3000 files for molecular networking, 5000 for library search

## Limitations

- ReDU only interacts with MassIVE; data and metadata from other resources are not yet incorporated
- Data are updated periodically, so recently contributed data will not display immediately in the File Selector
- No changelog found in the documentation repository, so vocabulary/template changes may be hard to track
- Recommended file limits (3000 molecular networking, 5000 library search) cap cohort size

## Evidence

- [readme] Unit tests cover the validators with naming conventions: [evidence span withheld — anchor status: external_doc]
- [readme] Validator distinguishes valid-but-unmatched data: [evidence span withheld — anchor status: external_doc]
