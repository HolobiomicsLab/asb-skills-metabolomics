---
name: cohort-group-assembly-for-library-search
description: Use when metadata-filtered public MS/MS files from ReDU must be assembled
  into cohorts for downstream GNPS spectral library search or molecular networking
  analysis.
when_to_use_negative:
- The analysis requires multiple groups assigned for library search — the documented
  constraint is that all selected files must be in Group 1 (G1) for that workflow
- The input is raw vendor-format mass spectrometry data rather than files already
  indexed in ReDU/MassIVE with ReDU sample information metadata
- More than 6 distinct cohorts are needed for molecular networking, exceeding the
  G1-G6 grouping support
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3314
tools:
- name: ReDU
  role: File Selector used to find files based on sample information (metadata) and
    assemble them into cohorts for GNPS analysis
  repo: https://github.com/mwang87/ReDU-MS2-GNPS
- name: MassIVE
  role: Public data repository whose files ReDU interacts with exclusively when assembling
    cohorts
- name: GNPS
  role: Downstream platform where assembled cohorts are analyzed via spectral library
    search (G1 only) or molecular networking (up to 6 groups)
  repo: https://github.com/CCMS-UCSD/GNPSDocumentation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-020-0916-7
    title: 'ReDU: a framework to find and reanalyze public mass spectrometry data'
  source_content_hash: sha256:9bd3122d44f7b092864db9e28767d575e0c2777311c264393f5ca37a41055fed
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3314
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cohort-group-assembly-for-library-search

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assembles publicly available tandem MS files selected via the ReDU File Selector into cohorts assigned to groups G1-G6, with the constraint that library search uses only Group 1 and molecular networking accepts at most 6 groups.

## When to use

Use when metadata-filtered public MS/MS files from ReDU must be assembled into cohorts for downstream GNPS spectral library search or molecular networking analysis.

## When NOT to use

- The analysis requires multiple groups assigned for library search — the documented constraint is that all selected files must be in Group 1 (G1) for that workflow
- The input is raw vendor-format mass spectrometry data rather than files already indexed in ReDU/MassIVE with ReDU sample information metadata
- More than 6 distinct cohorts are needed for molecular networking, exceeding the G1-G6 grouping support

## Inputs

- ReDU sample information (metadata) table in gnps_metadata.tsv column schema (Filename, MassIVE accession, SampleType, etc.)
- Attribute filter selections from the ReDU File Selector Attribute Filters Panel
- Selected public mass spectrometry filenames from MassIVE

## Outputs

- Manifest artifact mapping each selected filename to its group (G1-G6)
- Manifest listing applied metadata attribute filters
- Cohort grouping suitable for GNPS library search (G1 only) or molecular networking (up to G6)

## How to apply

Load the downloadable ReDU sample information table (gnps_metadata.tsv schema: Filename, MassIVE accession, SampleType, etc.) into pandas. Apply the Attribute Filters Panel semantics: select files whose metadata fields match chosen controlled-vocabulary attribute values, marking each active filter as a red-boxed attribute. Assemble the filtered files into groups G1 through G6, enforcing the documented constraint that library search in GNPS uses only Group 1 (G1), while molecular networking accepts at most 6 groups. Emit a manifest artifact mapping each selected filename to its group (G1-G6) and the applied metadata filters. Respect the recommended file limits of 3000 files for molecular networking and 5000 for library search; note that recently contributed data update periodically and may not display immediately.

## Related tools

- **ReDU** (File Selector used to find files based on sample information (metadata) and assemble them into cohorts for GNPS analysis) — https://github.com/mwang87/ReDU-MS2-GNPS
- **MassIVE** (Public data repository whose files ReDU interacts with exclusively when assembling cohorts)
- **GNPS** (Downstream platform where assembled cohorts are analyzed via spectral library search (G1 only) or molecular networking (up to 6 groups)) — https://github.com/CCMS-UCSD/GNPSDocumentation

## Examples

```
import pandas as pd; df = pd.read_csv('gnps_metadata.tsv', sep='\t'); g1 = df[df['SampleType'].isin(selected_values)]; g1[['Filename']].assign(group='G1').to_csv('cohort_manifest.tsv', sep='\t', index=False)
```

## Evaluation signals

- Manifest artifact exists and is a tab-separated table with Filename and group columns
- Manifest contains no more than 6 distinct groups
- No group is assigned to a library-search role other than G1
- Every filtered filename in the manifest has a non-empty group assignment from the G1-G6 set
- Applied metadata filters in the manifest match the chosen controlled-vocabulary attribute values

## Limitations

- Library search supports only a single group (G1); multi-group comparison is not available in that workflow
- Recommended file limits are 3000 for molecular networking and 5000 for library search
- Recently contributed data are updated periodically and will not display in the File Selector immediately
- ReDU only interacts with MassIVE; data from other repositories are not yet incorporated

## Evidence

- [abstract] File Selector enables users to find public mass spectrometry files based on sample information and assemble them into cohorts for GNPS analysis: "the ReDU File Selector is the tool that enables users to find files based on ReDU sample information (i.e. metadata) and assemble files into cohorts for analysis in GNPS"
- [abstract] Grouping constraints differ by workflow: "The recommended file limits are 3000 for molecular networking in GNPS and 5000 for library search in GNPS"
- [abstract] Attribute filters are visually marked in the File Selector: "If filter/s are used, they will appear as red box/boxes in the Attribute Filters Panel (upper-right corner) of the page"
