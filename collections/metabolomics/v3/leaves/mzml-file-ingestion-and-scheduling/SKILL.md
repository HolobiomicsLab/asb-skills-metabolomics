---
name: mzml-file-ingestion-and-scheduling
description: Use when an instrument PC writes vendor raw files that must be automatically
  converted to mzML, picked up by a monitoring daemon, and queued for QC processing
  without manual intervention.
when_to_use_negative:
- Input files are already processed feature tables or already converted, registered
  mzML files that QC4Metabolomics has previously ingested — ingestion and scheduling
  would duplicate records.
- The analysis goal is retrospective postanalysis correction of intensity drifts rather
  than real-time detection of newly acquired files.
- Data are not MS-based untargeted metabolomics acquisitions (e.g. non-MS omics or
  NMR data); the pipeline's modules assume chromatographic MS files.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0091
tools:
- name: QC4Metabolomics
  role: Hosts the continuous processing process whose module_Files and module_File_schedule
    modules detect, register, and schedule newly acquired mzML/mzXML/mzData files.
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Provides the docker-compose deployment (including the ProteoWizard converter)
    so that raw-to-mzML conversion and file ingestion run OS-agnostically on the network
    drive.
- name: R
  role: R package and Shiny web application in which module_Files and module_File_schedule
    are implemented and executed continuously.
- name: Shiny
  role: Web application front end through which the ingestion/scheduling configuration
    is deployed and monitored alongside demo data.
  repo: https://github.com/stanstrup/QC4Metabolomics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:3637f5c7951b37a5ed3991e750836c635db6001f4a7b462fb8e751f35dd1c35b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# mzml-file-ingestion-and-scheduling

## Summary

Continuously detect newly acquired metabolomics data files (mzML, mzXML, mzData), register them in a database, and schedule them for downstream quality-parameter extraction within the QC4Metabolomics processing loop.

## When to use

Use when an instrument PC writes vendor raw files that must be automatically converted to mzML, picked up by a monitoring daemon, and queued for QC processing without manual intervention.

## When NOT to use

- Input files are already processed feature tables or already converted, registered mzML files that QC4Metabolomics has previously ingested — ingestion and scheduling would duplicate records.
- The analysis goal is retrospective postanalysis correction of intensity drifts rather than real-time detection of newly acquired files.
- Data are not MS-based untargeted metabolomics acquisitions (e.g. non-MS omics or NMR data); the pipeline's modules assume chromatographic MS files.

## Inputs

- Vendor-specific instrument raw files (e.g. .raw) copied to an accessible network drive via a Windows batch script
- Converted mzML, mzXML, or mzData files placed in a watched directory
- Optional text file containing file paths of newly acquired files

## Outputs

- Entries in the QC4Metabolomics database file table registering each newly detected file
- Scheduled processing jobs queued in module_File_schedule for downstream modules (module_File_info, module_TrackCmp, module_Contaminants)
- Extracted metadata (instrument name, project name, ionization mode, sample name) per file once downstream modules run

## How to apply

Set up a Windows batch script on the instrument PC to copy vendor raw files to a network drive that QC4Metabolomics can access, then run ProteoWizard (provided in Docker, OS-agnostic) to convert vendor-specific raw data to mzML. Start the continuously running processing process so that module_Files finds newly acquired files (mzML, mzXML, mzData etc.) either by traversing a directory or by reading from a text file containing file paths, and adds them to a table in the database. module_File_schedule then checks whether new filenames were added to the database and schedules them for processing. Rationale: this ingestion-and-scheduling stage is what enables real-time QC, because downstream modules (module_File_info for metadata such as instrument name, project name, ionization mode and sample name, and module_TrackCmp for m/z, RT, intensity and peak-shape extraction) only run on files registered and scheduled by these modules.

## Related tools

- **QC4Metabolomics** (Hosts the continuous processing process whose module_Files and module_File_schedule modules detect, register, and schedule newly acquired mzML/mzXML/mzData files.) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Provides the docker-compose deployment (including the ProteoWizard converter) so that raw-to-mzML conversion and file ingestion run OS-agnostically on the network drive.)
- **R** (R package and Shiny web application in which module_Files and module_File_schedule are implemented and executed continuously.)
- **Shiny** (Web application front end through which the ingestion/scheduling configuration is deployed and monitored alongside demo data.) — https://github.com/stanstrup/QC4Metabolomics

## Examples

```
docker-compose up -d  # starts QC4Metabolomics so module_Files continuously scans the watched mzML directory and module_File_schedule queues new files
```

## Evaluation signals

- module_Files log shows each new mzML/mzXML/mzData file appearing in the database file table shortly after it is written to the watched directory or listed in the path text file.
- module_File_schedule marks every registered file as scheduled so that module_File_info and module_TrackCmp run on it without manual intervention.
- On the bundled demo data, each scheduled file proceeds through quality-parameter extraction in roughly 14 s and contaminant screening against the ~800-ion contaminant list in roughly 8 s, confirming ingestion did not stall the queue.
- Extracted metadata (instrument name, project name, ionization mode, sample name) is populated for every ingested file, demonstrating successful hand-off to module_File_info.
- Re-running module_Files over the same directory does not create duplicate database entries for identical filenames.

## Limitations

- Absolute file-detection and conversion speed depends on the host machine and network drive latency; the article reports per-file processing times (about 14 s extraction, about 8 s contaminant screening) only on its demo setup.
- module_Files relies on directory traversal or a text file of file paths — file paths that change location or are written with non-standard extensions (not mzML, mzXML, mzData etc.) will not be detected.
- Raw-to-mzML conversion depends on ProteoWizard via Docker; vendor formats not supported by ProteoWizard cannot be ingested automatically.

## Evidence

- [methods] A Windows batch script that is intended for use on the instrument PC to copy files to e.g. a network drive that QC4Metabolomics can access: "A Windows batch script that is intended for use on the instrument PC to copy files to e.g. a network drive that QC4Metabolomics can access"
- [methods] An automatic converter that uses ProteoWizard (in Docker and thus OS-agnostic) to convert vendor-specific raw data to mzML: "An automatic converter that uses ProteoWizard
,
 (in Docker and thus OS-agnostic) to convert vendor-specific raw
data to mzML"
- [methods] module_Files looks for newly acquired files (mzML, mzXML, mzData etc.) either by traversing a directory or by reading from a text file containing file paths and then adds them to a table in the: "module_Files looks for newly acquired
files (mzML, mzXML, mzData etc.) either by traversing a directory
or by reading from a text file containing file paths and then adds
them to a table in the"
- [methods] module_File_schedule checks if new filenames were added to the database: "module_File_schedule checks if new
filenames were added to the database"
- [abstract] Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data.: "Docker images and a docker-compose setup file are also provided for easy deployment, along with demo data."
