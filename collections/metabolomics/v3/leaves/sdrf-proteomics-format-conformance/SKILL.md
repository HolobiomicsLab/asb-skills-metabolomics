---
name: sdrf-proteomics-format-conformance
description: Use when raw MS files plus their associated metadata (LC column info,
  study factors, assay names, raw file names) are already stored in MSConnect's SQL
  database (e.g., deposited via the File Uploader) and a standardized, submission-ready
  SDRF-Proteomics file for PRIDE/MassIVE must be generated.
when_to_use_negative:
- The goal is the actual PRIDE or MassIVE upload/deposition rather than generating
  a format-conformant SDRF file — this skill covers format conformance only
- The sample metadata was never captured in MSConnect's SQL database (no File Uploader
  association exists), so there is nothing to map onto the SDRF schema
- The dataset is not MS-based proteomics/omics data managed by MSConnect (e.g., a
  feature table already outside MSConnect's pipeline)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MSConnect
  role: Source of the SQL-backed, web/programmatically searchable data structure linking
    users' raw MS files and sample metadata
  repo: https://github.com/RTKlab-BYU/MSConnect
- name: Raw_File_Uploader
  role: Captures the sample metadata associated with each raw MS file on upload, providing
    the fields to map onto the SDRF schema
  repo: https://github.com/RTKlab-BYU/Raw_File_Uploader
- name: Django
  role: Web framework behind MSConnect; its ORM/REST framework is used to query the
    SQL database for sample-metadata records
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00854
    title: 'MSConnect: Open-Source, End-to-End Platform for Automated Mass Spectrometry
      Data Management, Analysis, and Visualization'
  source_content_hash: sha256:a4c7d4dee5a6bde1597606883334ca435207bfa3a3b13363f66711b038d9a674
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/RTKlab-BYU/MSConnect
  tools:
  - Django
  - MSConnect
  - Raw_File_Uploader
  - SQL
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# sdrf-proteomics-format-conformance

## Summary

Generates a submission-ready SDRF-Proteomics data package (.sdrf.tsv) by mapping sample metadata captured in MSConnect's SQL-backed database onto the SDRF-Proteomics column schema, one row per raw MS file. This realizes, in advance of its release, the experimental-design module the MSConnect authors describe as under development.

## When to use

Use when raw MS files plus their associated metadata (LC column info, study factors, assay names, raw file names) are already stored in MSConnect's SQL database (e.g., deposited via the File Uploader) and a standardized, submission-ready SDRF-Proteomics file for PRIDE/MassIVE must be generated.

## When NOT to use

- The goal is the actual PRIDE or MassIVE upload/deposition rather than generating a format-conformant SDRF file — this skill covers format conformance only
- The sample metadata was never captured in MSConnect's SQL database (no File Uploader association exists), so there is nothing to map onto the SDRF schema
- The dataset is not MS-based proteomics/omics data managed by MSConnect (e.g., a feature table already outside MSConnect's pipeline)

## Inputs

- Sample-metadata records in the MSConnect SQL database (LC column info, study factors, project names) linked to raw MS files
- Raw MS file inventory uploaded via the Raw File Uploader (e.g., Orbitrap Astral, timsTOF Ultra, Orbitrap Exploris 480 acquisitions)
- SDRF-Proteomics column schema definition

## Outputs

- A single .sdrf.tsv file with an SDRF-style header line and one row per raw MS file
- Validation report confirming TSV parsability, row-count equality with sample-metadata records, and no empty required columns

## How to apply

Load the sample metadata that MSConnect's File Uploader associated with each raw MS file, using Python/Django's ORM (or the Django REST framework API) against the MSConnect server's SQL database, whose 'SQL database linked data structure allows users to search data and files through a web interface or programmatically'. Map each metadata field (factor value, assay name, raw file name, LC column, study factors) onto the SDRF-Proteomics column schema. Emit a single tab-separated .sdrf.tsv with one row per raw MS file and an SDRF-style header line containing the mapped metadata columns. Validate format conformance only: the file must parse as TSV, carry an SDRF header, contain exactly as many data rows as sample-metadata records in the SQL database, and have no empty required columns. Do not perform the repository upload itself — per the article, PRIDE/MassIVE submission is a separate future direction.

## Related tools

- **MSConnect** (Source of the SQL-backed, web/programmatically searchable data structure linking users' raw MS files and sample metadata) — https://github.com/RTKlab-BYU/MSConnect
- **Raw_File_Uploader** (Captures the sample metadata associated with each raw MS file on upload, providing the fields to map onto the SDRF schema) — https://github.com/RTKlab-BYU/Raw_File_Uploader
- **Django** (Web framework behind MSConnect; its ORM/REST framework is used to query the SQL database for sample-metadata records)

## Examples

```
python manage.py shell -c "from django.db import connection; rows=list(connection.cursor().execute('SELECT * FROM sample_metadata')); csv.writer(open('out.sdrf.tsv','w'),delimiter='\t').writerows(rows)"
```

## Evaluation signals

- The emitted .sdrf.tsv parses as a TSV with an SDRF-style header line
- The number of data rows equals the number of sample-metadata records in the MSConnect SQL database
- Every row corresponds to exactly one uploaded raw MS file
- No required SDRF columns (e.g., factor value, assay name, raw file name) are empty
- Format conformance confirmed without attempting repository upload (PRIDE/MassIVE submission is out of scope)

## Limitations

- MSConnect's formal experimental-design module is not yet released — 'we are developing a module that will formally track experimental design and seamlessly generate data packages' — so this skill must be built manually on top of the SQL metadata
- The paper's demonstration is on the PXD037527 single-cell proteomics dataset; metadata completeness for other study types is unverified
- MSConnect depends on third-party software and metadata captured at upload time; any metadata absent from the File Uploader cannot be recovered by this skill

## Evidence
