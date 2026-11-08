---
name: tab-separated-manifest-formatting
description: Use when filtering the ReDU sample-information table (gnps_metadata.tsv
  schema) by controlled-vocabulary attributes and grouping the selected MassIVE filenames
  into cohorts for GNPS library search or molecular networking.
when_to_use_negative:
- Single-file or self-contained analyses that do not require cohort grouping of public
  MassIVE files — no manifest is needed.
- Reanalysis of data originating outside MassIVE (e.g. MetaboLights-only imports),
  since ReDU only interacts with MassIVE.
- Library-search jobs needing more than 5000 files or more than one input group, which
  violate the documented G1-only constraint.
edam_operation: http://edamontology.org/operation_0335
tools:
- name: ReDU
  role: Source of the sample-information (metadata) table and File Selector grouping
    semantics (Attribute Filters Panel, G1-G6 cohort assembly).
  repo: https://github.com/mwang87/ReDU-MS2-GNPS
- name: GNPS
  role: 'Downstream consumer of the group manifest: grouped files are launched as
    GNPS library search (G1 only) or molecular networking (max 6 groups) tasks.'
  repo: https://ccms-ucsd.github.io/GNPSDocumentation/
- name: MassIVE
  role: Public repository supplying the MS2 filenames/MassIVE accessions that the
    manifest rows must resolve to.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41592-020-0916-7
    title: 'ReDU: a framework to find and reanalyze public mass spectrometry data'
  source_content_hash: sha256:e040f7764670acd05f6863544ea85f5a584e1702e3324c8f3bde6ac59e6d988b
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# tab-separated-manifest-formatting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Emits a tab-separated manifest that maps each ReDU-selected public MS2 filename to a cohort group (G1–G6) plus the metadata filters applied, honoring GNPS grouping constraints (library search uses only G1; molecular networking accepts at most 6 groups).

## When to use

Use when filtering the ReDU sample-information table (gnps_metadata.tsv schema) by controlled-vocabulary attributes and grouping the selected MassIVE filenames into cohorts for GNPS library search or molecular networking.

## When NOT to use

- Single-file or self-contained analyses that do not require cohort grouping of public MassIVE files — no manifest is needed.
- Reanalysis of data originating outside MassIVE (e.g. MetaboLights-only imports), since ReDU only interacts with MassIVE.
- Library-search jobs needing more than 5000 files or more than one input group, which violate the documented G1-only constraint.

## Inputs

- ReDU sample-information metadata table (gnps_metadata.tsv schema with Filename, MassIVE accession, SampleType columns)
- Attribute-filter selections over ReDU controlled-vocabulary fields (e.g. SampleType)
- Analysis role assignment (library search vs. molecular networking, up to 6 groups)

## Outputs

- Tab-separated manifest with Filename and group columns mapping each selected public MS2 file to G1-G6
- Record of the applied metadata attribute filters per selected file

## How to apply

Load the downloadable ReDU sample-information table (matching the gnps_metadata.tsv column schema: Filename, MassIVE accession, SampleType, etc.) into pandas. Apply the Attribute Filters Panel semantics: retain files whose metadata fields match the chosen controlled-vocabulary values, and record each active filter (conceptually the red-boxed attributes shown in the File Selector). Assign the filtered filenames to groups G1 through G6, enforcing the documented constraints that GNPS library search takes only Group 1 and molecular networking accepts at most 6 groups, while respecting the recommended file limits of 3000 files for molecular networking and 5000 for library search. Write the artifact as a tab-separated table with explicit Filename and group columns, mapping every selected filename to exactly one group (G1-G6). Validate before reuse: the file parses as TSV, contains no more than 6 distinct groups, assigns no library-search role to a group other than G1, and every Filename resolves to a MassIVE accession in the source metadata.

## Related tools

- **ReDU** (Source of the sample-information (metadata) table and File Selector grouping semantics (Attribute Filters Panel, G1-G6 cohort assembly).) — https://github.com/mwang87/ReDU-MS2-GNPS
- **GNPS** (Downstream consumer of the group manifest: grouped files are launched as GNPS library search (G1 only) or molecular networking (max 6 groups) tasks.) — https://ccms-ucsd.github.io/GNPSDocumentation/
- **MassIVE** (Public repository supplying the MS2 filenames/MassIVE accessions that the manifest rows must resolve to.)

## Examples

```
import pandas as pd; df=pd.read_csv('gnps_metadata.tsv',sep='\t'); df.assign(group='G1')[['Filename','group']].to_csv('manifest.tsv',sep='\t',index=False)
```

## Evaluation signals

- Manifest parses as tab-separated text with explicit Filename and group columns.
- Manifest contains at most 6 distinct group values, all within G1-G6.
- No file carrying a library-search role is assigned to any group other than G1.
- Group file counts stay within the recommended limits: 3000 files for GNPS molecular networking and 5000 for GNPS library search.
- Every manifest Filename matches a Filename/MassIVE accession present in the gnps_metadata.tsv-derived input table.

## Limitations

- ReDU only interacts with MassIVE; data and metadata from other repositories are not covered (MetaboLights imports may carry 'ML import: not available' annotations).
- Recently contributed data are updated periodically and will not display recently contributed data immediately, so freshly submitted files may be absent from the metadata table.
- Library search via the File Selector supports only a single group (G1); multi-group cohort comparison requires molecular networking workflows.

## Evidence

- [readme] invaliddata_* - These are valid files that pass validation but fail to match with data in MassIVE: [evidence span withheld — anchor status: not_found]
