---
name: cohort-group-assembly-for-molecular-networking
description: Use when a repository-scale re- or co-analysis requires selecting public
  MS2 files by metadata attributes (SampleType, etc.) and grouping them into cohorts
  for GNPS spectral library search or molecular networking.
when_to_use_negative:
- Input is already a feature/quantification table or completed GNPS annotations rather
  than raw public MS2 file selection
- A library-search workflow that needs more than one comparison group (only G1 is
  supported for library search)
- 'Data or metadata not present in MassIVE/ReDU, since ''ReDU only interacts with
  MassIVE'' and imported data (e.g. MetaboLights) may be marked ''ML import: not available'''
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3339
tools:
- name: ReDU
  role: File Selector used to find public MS2 files by sample-information metadata
    and assemble them into groups G1–G6
  repo: https://github.com/mwang87/ReDU-MS2-GNPS
- name: GNPS
  role: Downstream platform receiving assembled cohorts for spectral library search
    and molecular networking
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
- name: MassIVE
  role: Public mass spectrometry data repository from which ReDU File Selector draws
    files and MassIVE accessions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-020-0916-7
    title: 'ReDU: a framework to find and reanalyze public mass spectrometry data'
  source_content_hash: sha256:bfddd8a4afdf5aa3da6c173b9fd8751f2f97af682b6d10f7bd3d1128c3666471
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3339
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cohort-group-assembly-for-molecular-networking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles publicly available tandem MS files into analysis cohorts (groups G1–G6) using ReDU sample-information metadata and its Attribute Filters Panel, producing a filename-to-group manifest ready for GNPS library search or molecular networking.

## When to use

Use when a repository-scale re- or co-analysis requires selecting public MS2 files by metadata attributes (SampleType, etc.) and grouping them into cohorts for GNPS spectral library search or molecular networking.

## When NOT to use

- Input is already a feature/quantification table or completed GNPS annotations rather than raw public MS2 file selection
- A library-search workflow that needs more than one comparison group (only G1 is supported for library search)
- Data or metadata not present in MassIVE/ReDU, since 'ReDU only interacts with MassIVE' and imported data (e.g. MetaboLights) may be marked 'ML import: not available'

## Inputs

- ReDU sample information (metadata) table (gnps_metadata.tsv-style, with Filename, MassIVE accession, SampleType columns)
- Chosen controlled-vocabulary attribute filter values from the ReDU Attribute Filters Panel
- Target GNPS workflow type (library search vs molecular networking) determining grouping rules

## Outputs

- Tab-separated cohort manifest mapping each selected Filename to its group (G1–G6) and the applied metadata filters
- Cohort assembly ready for GNPS spectral library search or molecular networking submission

## How to apply

Load the downloadable ReDU sample-information (metadata) table into pandas using its gnps_metadata.tsv column schema (Filename, MassIVE accession, SampleType, etc.). Apply the Attribute Filters Panel semantics: keep only files whose metadata fields match the chosen controlled-vocabulary attribute values, and mark each active filter as a red-boxed attribute. Assign each filtered filename to one of at most six groups (G1–G6); for GNPS spectral library search, every selected file must be placed in Group 1 (G1) because that workflow supports only a single group, while molecular networking accepts up to 6 groups and recommends file limits of 3000 (networking) and 5000 (library search). Emit a tab-separated manifest mapping each filename to its group and the applied filters, since ReDU serves as the launchpad into GNPS workflows. Validate the manifest before handing it to GNPS.

## Related tools

- **ReDU** (File Selector used to find public MS2 files by sample-information metadata and assemble them into groups G1–G6) — https://github.com/mwang87/ReDU-MS2-GNPS
- **GNPS** (Downstream platform receiving assembled cohorts for spectral library search and molecular networking) — https://github.com/CCMS-UCSD/GNPSDocumentation
- **MassIVE** (Public mass spectrometry data repository from which ReDU File Selector draws files and MassIVE accessions)

## Examples

```
import pandas as pd; df = pd.read_csv("gnps_metadata.tsv", sep="\t"); cohort = df[df.SampleType=="human"]; cohort["group"]="G1"; cohort.to_csv("cohort_manifest.tsv", sep="\t", index=False)
```

## Evaluation signals

- Manifest exists and is a tab-separated table with Filename and group columns
- Manifest contains no more than 6 distinct groups (G1–G6)
- No group is assigned to a library-search role other than G1 (all library-search files in Group 1)
- Every selected file's metadata fields match the chosen controlled-vocabulary attribute values, and each active filter is recorded as a red-boxed attribute
- Cohort size respects recommended GNPS file limits: 3000 for molecular networking and 5000 for library search

## Limitations

- ReDU only interacts with MassIVE; data from other repositories (e.g. MetaboLights imports) may be incomplete
- ReDU data are updated periodically and will not display recently contributed data immediately
- File limits constrain cohort size (3000 recommended for molecular networking, 5000 for library search)
- Library search via ReDU File Selector does not support multiple groups; only Group 1 (G1) can be used

## Evidence

- [readme] README manual validation includes File Selector counts: [evidence span withheld — anchor status: external_doc]
