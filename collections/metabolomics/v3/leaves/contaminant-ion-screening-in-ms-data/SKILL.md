---
name: contaminant-ion-screening-in-ms-data
description: Use when mzML/mzXML/mzData files from untargeted MS-based metabolomics
  acquisition must be checked for known background contaminants — e.g. to diagnose
  carryover, contamination from labware, or user-defined compound exposure.
when_to_use_negative:
- The input is already a curated feature table or blank-subtracted peak list rather
  than raw mzML/mzXML vendor output — QC4Metabolomics expects raw file traversal/ingestion
  via module_Files.
- The goal is drift correction or intensity normalization (e.g. LOESS on pooled QC
  samples) rather than detecting discrete contaminant ions; QC4Metabolomics monitors
  contaminants but does not perform postanalysis intensity-drift correction itself.
- Target analytes are rare or user-specific compounds not represented in the ~800-contaminant
  library and no user-defined contaminant list has been supplied — screening will
  return nothing actionable.
edam_operation: http://edamontology.org/operation_3214
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: QC4Metabolomics
  role: Supplies module_Contaminants (screening against ~800 known contaminant ions)
    plus module_Files/module_File_schedule for file ingestion and module_Warner for
    threshold alerting
  repo: https://github.com/stanstrup/QC4Metabolomics
- name: Docker
  role: Provides the docker-compose deployment used to run QC4Metabolomics and its
    ProteoWizard-based mzML conversion pipeline
- name: ProteoWizard
  role: Vendor-specific raw data to mzML converter invoked automatically by the QC4Metabolomics
    Docker setup
- name: R
  role: Implementation language for the QC4Metabolomics package and its module_Contaminants
    screening module
- name: Shiny
  role: Web dashboard where contaminant levels are presented for real-time or retrospective
    monitoring
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c07078
    title: 'QC4Metabolomics: Real-Time and Retrospective Quality Control of Metabolomics
      Data'
  source_content_hash: sha256:932069bbdc62a912f02159cb20fab18686634b5ab297a5d2c1958989b0183614
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
  edam_operation: http://edamontology.org/operation_3214
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# contaminant-ion-screening-in-ms-data

## Summary

Screens each acquired metabolomics LC-MS file against a built-in library of nearly 800 known contaminant ions to detect contaminant carryover or exposure via QC4Metabolomics, flagging files whose contaminant signal levels exceed user-defined thresholds.

## When to use

Use when mzML/mzXML/mzData files from untargeted MS-based metabolomics acquisition must be checked for known background contaminants — e.g. to diagnose carryover, contamination from labware, or user-defined compound exposure.

## When NOT to use

- The input is already a curated feature table or blank-subtracted peak list rather than raw mzML/mzXML vendor output — QC4Metabolomics expects raw file traversal/ingestion via module_Files.
- The goal is drift correction or intensity normalization (e.g. LOESS on pooled QC samples) rather than detecting discrete contaminant ions; QC4Metabolomics monitors contaminants but does not perform postanalysis intensity-drift correction itself.
- Target analytes are rare or user-specific compounds not represented in the ~800-contaminant library and no user-defined contaminant list has been supplied — screening will return nothing actionable.

## Inputs

- mzML (or mzXML/mzData) files from untargeted MS-based metabolomics runs, exposed to QC4Metabolomics via a watched directory or a text file of file paths
- built-in list of nearly 800 known contaminant ions (optionally extended with a user-defined contaminant list)

## Outputs

- Per-file contaminant detection records (contaminant m/z, retention time, intensity) written to the QC4Metabolomics database
- Dashboard visualizations of contaminant levels over time, with optional email warnings from module_Warner when a contaminant quality parameter exceeds a user-defined threshold

## How to apply

Convert vendor raw files to mzML with ProteoWizard, then let QC4Metabolomics's module_Files detect the newly acquired files and module_File_schedule add them to the processing database. The module_Contaminants step screens each scheduled file against the built-in list of nearly 800 known contaminant ions, extracting the detected m/z, retention time, intensity and peak-shape diagnostics for each contaminant. Results are presented on the Shiny web dashboard, where levels of common or user-defined contaminants can be monitored in (near) real-time as files are acquired or retrospectively on historic batches. Rationale: contaminant screening runs independently of quality-parameter extraction and takes roughly 8 s per file on the bundled demo data, making it cheap enough to run continuously during acquisition.

## Related tools

- **QC4Metabolomics** (Supplies module_Contaminants (screening against ~800 known contaminant ions) plus module_Files/module_File_schedule for file ingestion and module_Warner for threshold alerting) — https://github.com/stanstrup/QC4Metabolomics
- **Docker** (Provides the docker-compose deployment used to run QC4Metabolomics and its ProteoWizard-based mzML conversion pipeline)
- **ProteoWizard** (Vendor-specific raw data to mzML converter invoked automatically by the QC4Metabolomics Docker setup)
- **R** (Implementation language for the QC4Metabolomics package and its module_Contaminants screening module)
- **Shiny** (Web dashboard where contaminant levels are presented for real-time or retrospective monitoring)

## Examples

```
docker-compose up -d  # deploy QC4Metabolomics with demo data; module_Contaminants screens each mzML file against the built-in ~800 contaminant ions
```

## Evaluation signals

- Per-file contaminant-screening wall-clock time is of the same order as the reported ~8 s on demo data (absolute values depend on host hardware).
- Every newly ingested mzML file has contaminant records in the QC4Metabolomics database (no silently skipped files after module_File_schedule processing).
- Contaminant hits carry m/z, RT and intensity values consistent with the user-defined or built-in contaminant ion list entries.
- module_Warner fires only when a contaminant quality parameter exceeds a user-defined threshold, with no spurious alerts on clean files.
- Contaminant signal trends (intensity over time) are consistent across files from the same batch and visibly change when contamination is introduced.

## Limitations

- Only detects contaminants represented in the built-in ~800-ion list or a supplied user-defined list — novel or unexpected contaminants are missed unless added manually.
- Contaminant screening adds ~8 s per file on the demo dataset; very high sample throughput may require tuning of the continuously running processing process.
- The built-in contaminant list is fixed; filtering out nonobvious contaminants or ion-suppression artifacts in specific chromatographic regions is not automated and must be judged manually on the dashboard.

## Evidence

- [methods] built-in list of nearly 800 known contaminant ions: "module_Contaminants uses a list of nearly 800 known contaminant ions"
- [abstract] contaminant levels monitoring: "QC4Metabolomics also monitors the levels of common or user-defined contaminants."
- [methods] email warning threshold: "module_Warner can send emails warning when a quality parameter exceeds a user defined threshold"
- [full_text] per-file screening timing: [evidence span withheld — anchor status: not_found]
- [methods] vendor raw to mzML conversion: [evidence span withheld — anchor status: not_found]
- [abstract] dashboard presentation: "and presents the results on a web dashboard"
- [abstract] screening in real-time or retrospective mode: "QC4Metabolomics monitors files as they are acquired or retrospectively by tracking any user-defined compound(s)"
