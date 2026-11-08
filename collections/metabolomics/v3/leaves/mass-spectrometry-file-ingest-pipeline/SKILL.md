---
name: mass-spectrometry-file-ingest-pipeline
description: Use when new instrument acquisition files (mzML, mzXML, mzData) must
  flow continuously from an instrument PC into a QC-metrics database and dashboard
  with no manual file handling.
when_to_use_negative:
- Data is already registered in the QC4Metabolomics database or already processed
  into tracked peaks — rerunning ingest would duplicate files
- Goal is retrospective analysis of a single already-converted mzML file rather than
  continuous automated ingest of newly acquired files
- Vendor raw format is needed for the analysis and lossless conversion to mzML is
  unacceptable
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: QC4Metabolomics
  role: Hosts module_Files, which detects newly acquired mzML/mzXML/mzData files and
    adds them to the database
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: ProteoWizard
  role: Automatic converter, run in Docker and thus OS-agnostic, converting vendor-specific
    raw data to mzML upstream of ingest
- name: Docker
  role: Packaging for the converter and full deployment via docker-compose for easy,
    OS-agnostic setup
- name: Shiny
  role: Web dashboard process that consumes the ingested files' extracted quality
    metrics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:113ebdcc33e7bb06f9f57f6acdec272267d2a042a97febd649f2468a1829ec8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/stanstrup/QC4Metabolomics
  tools:
  - Docker
  - ProteoWizard
  - QC4Metabolomics
  - R
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# mass-spectrometry-file-ingest-pipeline

## Summary

Automated ingest pipeline that moves vendor raw MS files from an instrument PC into a QC monitoring system: a Windows batch script copies raw files to network storage, ProteoWizard (Dockerized, OS-agnostic) converts them to mzML, and module_Files detects new mzML/mzXML/mzData files and registers them in a database so downstream QC modules can process them without operator intervention.

## When to use

Use when new instrument acquisition files (mzML, mzXML, mzData) must flow continuously from an instrument PC into a QC-metrics database and dashboard with no manual file handling.

## When NOT to use

- Data is already registered in the QC4Metabolomics database or already processed into tracked peaks — rerunning ingest would duplicate files
- Goal is retrospective analysis of a single already-converted mzML file rather than continuous automated ingest of newly acquired files
- Vendor raw format is needed for the analysis and lossless conversion to mzML is unacceptable

## Inputs

- Vendor-specific raw mass spectrometry files copied from the instrument PC
- mzML / mzXML / mzData converted files
- Optional text file containing file paths to ingest
- Watched directory on network storage accessible to QC4Metabolomics

## Outputs

- Rows in a mariaDB table registering each newly detected file
- Extracted per-file metadata (instrument name, project name, ionization mode, sample name) for the dashboard
- Entries consumed by module_File_schedule to trigger downstream QC processing (module_TrackCmp, module_Contaminants)

## How to apply

Set up the upstream transfer with a Windows batch script on the instrument PC that copies vendor raw files to a network drive the QC system can access, then run an automatic ProteoWizard-based converter in Docker to convert vendor-specific raw data to mzML. Run module_Files continuously in a fixed processing process, pointing it either at the watched directory (traverse it for new files) or at a text file of file paths, and have it insert each newly acquired file into a table in the mariaDB database. Downstream, module_File_schedule checks for new filenames in the database and module_File_info extracts metadata (instrument name, project name, ionization mode, sample name), which is what allows quality metrics to appear in the Shiny dashboard without an operator. Deploy the whole stack via the provided Docker images and docker-compose setup with environment-variable configuration so the converter is OS-agnostic and the pipeline restarts cleanly.

## Related tools

- **QC4Metabolomics** (Hosts module_Files, which detects newly acquired mzML/mzXML/mzData files and adds them to the database) — https://github.com/stanstrup/QC4Metabolomics
- **ProteoWizard** (Automatic converter, run in Docker and thus OS-agnostic, converting vendor-specific raw data to mzML upstream of ingest)
- **Docker** (Packaging for the converter and full deployment via docker-compose for easy, OS-agnostic setup)
- **Shiny** (Web dashboard process that consumes the ingested files' extracted quality metrics)

## Examples

```
docker-compose up -d  # starts the file-watcher process (module_Files), mariaDB, and the Shiny dashboard; drop converted mzML files into the watched directory
```

## Evaluation signals

- Every new mzML/mzXML/mzData file in the watched directory (or listed in the path text file) appears as a row in the database table with no operator action
- module_Log shows the ingest activities of module_Files and module_File_schedule for each new file
- module_File_info successfully extracts instrument name, project name, ionization mode, and sample name from each ingested file
- Converted mzML files open and are processable by the downstream peak-tracking modules (module_TrackCmp against user-defined m/z and RT pairs)
- The docker-compose deployment starts both fixed processes (file-watcher and Shiny dashboard) with the mariaDB database connecting them

## Limitations

- File formats are limited to those module_Files recognizes (mzML, mzXML, mzData, etc.); exotic vendor formats must be converted via ProteoWizard first
- Instrument PC must run the Windows batch script to copy files to a location QC4Metabolomics can access; the pipeline silently stalls if that copy step stops
- Ingest alone produces no quality judgment — it only registers files; quality signals come from downstream modules (module_TrackCmp, module_Contaminants, module_Warner)

## Evidence

- [methods] module_Files detects and registers new files: "module_Files looks for newly acquired files (mzML, mzXML, mzData etc.) either by traversing a directory or by reading from a text file containing file paths and then adds them to a table in the"
- [methods] ProteoWizard conversion step: [evidence span withheld — anchor status: not_found]
- [methods] Upstream file copy from instrument PC: "A Windows batch script that is intended for use on the instrument PC to copy files to e.g. a network drive that QC4Metabolomics can access"
- [readme] Documentation pointer in README: "Please refer to the documentation here: http://stanstrup.github.io/QC4Metabolomics/"
