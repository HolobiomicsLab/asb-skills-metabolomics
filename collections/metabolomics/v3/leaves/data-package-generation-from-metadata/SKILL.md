---
name: data-package-generation-from-metadata
description: Use when preparing a PRIDE/MassIVE submission package (e.g., SDRF-Proteomics
  .sdrf.tsv) from raw-MS-file sample metadata already managed in MSConnect's SQL-backed
  database.
when_to_use_negative:
- The goal is the actual repository upload itself (PRIDE/MassIVE submission) — this
  skill only covers generating a conformant package, and the article treats repository
  upload automation as a separate discussion point
- The raw MS files lack associated metadata records in the MSConnect SQL database
  (e.g., files uploaded outside the File Uploader), so the required SDRF columns cannot
  be populated
- The input is already an SDRF-formatted file that only needs re-validation rather
  than generation from metadata
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0092
tools:
- name: MSConnect
  role: Hosts the SQL database linking raw MS files and sample metadata; queried via
    Django ORM/REST framework to source the SDRF rows
  repo: https://github.com/RTKlab-BYU/MSConnect
- name: Raw_File_Uploader
  role: 'Populates the metadata: resides on the mass spectrometer control computer
    and sends MS data files and associated metadata to the MSConnect Server'
  repo: https://github.com/RTKlab-BYU/Raw_File_Uploader
- name: Django
  role: Web framework backing MSConnect's SQL data structure; Django ORM queries retrieve
    the per-file metadata for mapping onto SDRF columns
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1021/acs.jproteome.4c00854
    title: 'MSConnect: Open-Source, End-to-End Platform for Automated Mass Spectrometry
      Data Management, Analysis, and Visualization'
  source_content_hash: sha256:e77be11bc133c0a4ad2587d02dd108c9f7bb417ff34076838b1e8f303b580e5d
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
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# data-package-generation-from-metadata

## Summary

Generates a submission-ready experimental-design data package (SDRF-Proteomics-style TSV, one row per raw MS file) by mapping sample metadata stored in MSConnect's SQL-backed Django database onto standardized submission columns. Grounded in the MSConnect roadmap, which reserves experimental-design tracking and data-package generation as a sanctioned module of the platform.

## When to use

Use when preparing a PRIDE/MassIVE submission package (e.g., SDRF-Proteomics .sdrf.tsv) from raw-MS-file sample metadata already managed in MSConnect's SQL-backed database.

## When NOT to use

- The goal is the actual repository upload itself (PRIDE/MassIVE submission) — this skill only covers generating a conformant package, and the article treats repository upload automation as a separate discussion point
- The raw MS files lack associated metadata records in the MSConnect SQL database (e.g., files uploaded outside the File Uploader), so the required SDRF columns cannot be populated
- The input is already an SDRF-formatted file that only needs re-validation rather than generation from metadata

## Inputs

- Sample metadata records in the MSConnect server's SQL-backed database (populated by Raw_File_Uploader with LC column info, study factors, project names, raw file names)
- Raw MS files uploaded to MSConnect (e.g., from Orbitrap Astral, timsTOF Ultra, or Orbitrap Exploris 480 acquisitions)
- SDRF-Proteomics column schema (factor value, assay name, raw file name fields)

## Outputs

- A single SDRF-formatted .sdrf.tsv file with an SDRF-style header line and one row per raw MS file, carrying mapped sample metadata columns
- A validated, submission-ready data package suitable as the basis for PRIDE/MassIVE record upload

## How to apply

Load the sample metadata that MSConnect's File Uploader associated with the uploaded raw MS files (LC column info, study factors, project names) via Python/Django ORM queries against the MSConnect server's SQL database, which links users' data and files for web/programmatic search. Map each metadata field (factor value, assay name, raw file name) onto the SDRF-Proteomics column schema. Emit a single .sdrf.tsv file containing one row per raw MS file with the mapped metadata columns, mirroring what the paper's planned experimental design module would 'seamlessly generate from data packages'. Accept the output only if it parses as a TSV with an SDRF-style header line, one data row per uploaded raw file, and no empty required columns — the paper demonstrated the metadata pipeline on the PXD037527 single-cell proteomics dataset deposited to ProteomeXchange via PRIDE.

## Related tools

- **MSConnect** (Hosts the SQL database linking raw MS files and sample metadata; queried via Django ORM/REST framework to source the SDRF rows) — https://github.com/RTKlab-BYU/MSConnect
- **Raw_File_Uploader** (Populates the metadata: resides on the mass spectrometer control computer and sends MS data files and associated metadata to the MSConnect Server) — https://github.com/RTKlab-BYU/Raw_File_Uploader
- **Django** (Web framework backing MSConnect's SQL data structure; Django ORM queries retrieve the per-file metadata for mapping onto SDRF columns)

## Examples

```
python -c "import django; django.setup(); import pandas as pd; from msconnect.models import SampleMetadata; pd.DataFrame.from_records(SampleMetadata.objects.values()).to_csv('PXD037527.sdrf.tsv', sep='\t', index=False)"
```

## Evaluation signals

- The emitted .sdrf.tsv parses as a well-formed TSV with an SDRF-style header line
- The number of data rows equals the number of sample-metadata records in the MSConnect SQL database
- Every required SDRF column (e.g., raw file name, factor value, assay name) is non-empty in all rows
- Each row corresponds to exactly one uploaded raw MS file (no missing or duplicated files)
- Spot-check a subset of rows against the source metadata (as demonstrated on the PXD037527 single-cell proteomics dataset)

## Limitations

- MSConnect's experimental design module is not yet released — the article states the module is still being developed, so package generation currently relies on custom Python/Django queries rather than a sanctioned MSConnect feature
- MSConnect is not a standalone analysis tool and depends on third-party software; the generated package does not itself perform repository upload
- The MSConnect repository has no changelog, so tracking schema/API changes affecting metadata fields requires manual vigilance

## Evidence

- [discussion] Experimental design module roadmap: "we are developing a module that will formally track experimental design and seamlessly generate data packages"
- [methods] Metadata provenance from File Uploader: "The File Uploader software resides on the control computer of the mass spectrometer and sends MS data files and associated metadata to the MSConnect Server"
- [readme] Programmatic access to linked data: "A SQL database linked data structure that allows users to search data and files through a web interface or programmatically."
- [methods] Target repositories and reference dataset: "deposited to the ProteomeXchange Consortium via the PRIDE partner repository29 with the data set identifier PXD037527"
