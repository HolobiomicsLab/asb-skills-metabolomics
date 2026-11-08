---
name: experimental-design-tracking
description: Use when raw MS files and their sample metadata (LC column info, study
  factors, project names) already reside in an MSConnect server database and you need
  a standardized, submission-ready experimental-design data package rather than an
  ad-hoc sample sheet.
when_to_use_negative:
- The goal is the actual upload of records to PRIDE or MassIVE — this skill validates
  SDRF format conformance only, not repository submission
- The data are not managed in MSConnect (e.g. no SQL-backed database linking raw MS
  files to their File Uploader metadata) — the ORM-based mapping has no source to
  read from
- MSConnect is being used as a standalone analysis tool — per the README it 'is not
  a standalone data analysis tool' and depends on third-party software for data processing/analysis
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: MSConnect
  role: Web-based platform whose SQL database links users' data and files, providing
    the sample metadata records that are mapped onto the SDRF schema
  repo: https://github.com/RTKlab-BYU/MSConnect
- name: Raw File Uploader
  role: Resides on the mass spectrometer control computer and uploads raw MS data
    files together with their associated metadata to the MSConnect Server, creating
    the records this skill reads
  repo: https://github.com/RTKlab-BYU/Raw_File_Uploader
- name: Django
  role: Web framework backing MSConnect's SQL database; its ORM (and Django REST framework
    API) is used to query sample-metadata records programmatically
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00854
    title: 'MSConnect: Open-Source, End-to-End Platform for Automated Mass Spectrometry
      Data Management, Analysis, and Visualization'
  source_content_hash: sha256:df1f8537a419dc7deac30ff93641734d71921d37b8b0001b7a0a1ee49219b5f6
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/RTKlab-BYU/MSConnect
  tools:
  - Django
  - MSConnect
  - Raw File Uploader
  - SQL
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# experimental-design-tracking

## Summary

Formally track experimental design for MS-based proteomics by mapping sample metadata stored in MSConnect's SQL-backed database onto the SDRF-Proteomics column schema, emitting a submission-ready .sdrf.tsv with one row per raw MS file for PRIDE/MassIVE submission packages.

## When to use

Use when raw MS files and their sample metadata (LC column info, study factors, project names) already reside in an MSConnect server database and you need a standardized, submission-ready experimental-design data package rather than an ad-hoc sample sheet.

## When NOT to use

- The goal is the actual upload of records to PRIDE or MassIVE — this skill validates SDRF format conformance only, not repository submission
- The data are not managed in MSConnect (e.g. no SQL-backed database linking raw MS files to their File Uploader metadata) — the ORM-based mapping has no source to read from
- MSConnect is being used as a standalone analysis tool — per the README it 'is not a standalone data analysis tool' and depends on third-party software for data processing/analysis

## Inputs

- Sample metadata records stored in the MSConnect server's SQL-backed database (LC column info, study factors, project names) linked to uploaded raw MS files
- Raw MS file inventory as registered by the Raw File Uploader on the instrument control computer
- SDRF-Proteomics column schema (factor value, assay name, raw file name, etc.)

## Outputs

- A single submission-ready SDRF-Proteomics file (.sdrf.tsv) with one data row per raw MS file and mapped metadata columns

## How to apply

Access the MSConnect server's SQL database via Python/Django's ORM, reading the sample metadata records that the File Uploader associated with each uploaded raw MS file. Map each metadata field (factor value, assay name, raw file name) onto the SDRF-Proteomics column schema, using MSConnect's SQL-linked data structure that supports programmatic search of data and files. Emit a single SDRF-formatted file (e.g. an .sdrf.tsv) containing one row per raw MS file with the mapped metadata columns, mirroring what the paper's unreleased experimental-design module would 'seamlessly generate from data packages'. Note this skill validates format conformance only — it does not perform the PRIDE/MassIVE repository upload itself, and the demonstration case is the PXD037527 single-cell proteomics dataset.

## Related tools

- **MSConnect** (Web-based platform whose SQL database links users' data and files, providing the sample metadata records that are mapped onto the SDRF schema) — https://github.com/RTKlab-BYU/MSConnect
- **Raw File Uploader** (Resides on the mass spectrometer control computer and uploads raw MS data files together with their associated metadata to the MSConnect Server, creating the records this skill reads) — https://github.com/RTKlab-BYU/Raw_File_Uploader
- **Django** (Web framework backing MSConnect's SQL database; its ORM (and Django REST framework API) is used to query sample-metadata records programmatically)

## Examples

```
python manage.py shell -c "from msconnect.models import RawFile; import csv; rows=RawFile.objects.values('name','factor_value','assay'); w=csv.writer(open('submission.sdrf.tsv','w'),delimiter='\t'); w.writerow(SDRF_HEADER); w.writerows(rows)"
```

## Evaluation signals

- The emitted .sdrf.tsv parses as a valid TSV with an SDRF-style header line
- The number of data rows in the .sdrf.tsv equals the number of sample-metadata records in the MSConnect SQL database (one row per raw MS file)
- No required SDRF columns are empty in the emitted file
- Round-trip check: each row's raw file name matches a file registered by the File Uploader in the MSConnect server
- Reproduce the demonstration on the PXD037527 single-cell proteomics dataset as a ground-truth case

## Limitations

- The MSConnect authors' native experimental-design module is not yet released ('we are developing a module that will formally track experimental design and seamlessly generate data packages'), so this skill re-implements the mapping externally via the Django ORM
- Completeness depends on what the File Uploader captured — missing factor values or assay names in the source metadata will surface as empty required SDRF columns
- MSConnect repository has no changelog, so schema/database field names may drift between versions without a documented record

## Evidence

- [discussion] Basis for the skill: the sanctioned future module: "we are developing a module that will formally track experimental design and seamlessly generate data packages"
- [methods] Origin of the metadata records being mapped: "The File Uploader software resides on the control computer of the mass spectrometer and sends MS data files and associated metadata to the MSConnect Server"
- [readme] Programmatic access path to the metadata: "A SQL database linked data structure that allows users to search data and files through a web interface or programmatically."
- [methods] Demonstration dataset: "deposited to the ProteomeXchange Consortium via the PRIDE partner repository29 with the data set identifier PXD037527"
