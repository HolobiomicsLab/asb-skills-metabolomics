---
name: hmdb-data-ingestion
description: Use when a metabolomics annotation workflow needs a local, reproducible
  reference database of compounds plus experimental MS2 spectra built from an HMDB
  release, rather than querying HMDB online.
when_to_use_negative:
- When an existing CompDb.Hsapiens.HMDB.5.0.sqlite is already available — the correct
  step is to load it with CompDb(), not rebuild it from HMDB sources.
- When the goal is to annotate experimental spectra against a reference (MS2 spectral
  matching via matchSpectra); this skill only creates the reference database, it does
  not perform matching.
- When the annotation target database is not HMDB (e.g. MassBank or GNPS-derived libraries);
  other backends such as MsBackendMassbank or MsBackendMgf cover those formats.
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CompoundDb
  role: Creates and manages the CompDb reference database; ingests HMDB compound records
    and experimental MS2 spectra into a portable SQLite CompDb file with embedded
    source/version metadata.
  repo: https://rformassspectrometry.github.io/CompoundDb/
- name: R
  role: Execution environment for building and loading the CompDb database (via Bioconductor/BiocManager
    installation of CompoundDb).
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:0fd0792038895eec70ad89fb60d53a6661d8f9d0e32d89cc0d53b5d218e53d0e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CompoundDb
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# hmdb-data-ingestion

## Summary

Builds a portable, self-contained CompoundDb (CompDb) SQLite reference annotation database from the Human Metabolome Database (HMDB) version 5.0 data release, containing compound annotations and experimental (non-predicted) MS2 spectra for downstream MS1/MS2 metabolomics annotation in R.

## When to use

Use when a metabolomics annotation workflow needs a local, reproducible reference database of compounds plus experimental MS2 spectra built from an HMDB release, rather than querying HMDB online.

## When NOT to use

- When an existing CompDb.Hsapiens.HMDB.5.0.sqlite is already available — the correct step is to load it with CompDb(), not rebuild it from HMDB sources.
- When the goal is to annotate experimental spectra against a reference (MS2 spectral matching via matchSpectra); this skill only creates the reference database, it does not perform matching.
- When the annotation target database is not HMDB (e.g. MassBank or GNPS-derived libraries); other backends such as MsBackendMassbank or MsBackendMgf cover those formats.

## Inputs

- HMDB version 5.0 data release (2021-11-02) downloaded from hmdb.ca, containing all compound/metabolite records
- Experimental MS2 spectra from the same HMDB release (predicted/in-silico spectra must be excluded before ingestion)

## Outputs

- CompDb.Hsapiens.HMDB.5.0.sqlite — a portable CompDb SQLite database with 217,776 compound annotations and 64,920 experimental MS2 spectra
- A CompDb object loadable in R for use as an MS2 reference library in MetaboAnnotation workflows

## How to apply

Download the HMDB version 5.0 data release (2 November 2021) containing all compound/metabolite records and experimental MS2 spectra from hmdb.ca. Build the CompDb SQLite database with the CompoundDb package, explicitly excluding predicted (in silico) spectra and embedding source/version metadata so the build is reproducible. The finished database is a single portable file, named CompDb.Hsapiens.HMDB.5.0.sqlite, which can then be loaded in R via CompDb(dbname = ...) and inspected through its compound and MS2 spectrum tables. Validate the build by checking that the compound table row count equals 217,776 and the experimental MS2 spectrum table row count equals 64,920; this database then serves as the MS2 reference library for matchSpectra-based annotation in the MetaboAnnotation ecosystem.

## Related tools

- **CompoundDb** (Creates and manages the CompDb reference database; ingests HMDB compound records and experimental MS2 spectra into a portable SQLite CompDb file with embedded source/version metadata.) — https://rformassspectrometry.github.io/CompoundDb/
- **R** (Execution environment for building and loading the CompDb database (via Bioconductor/BiocManager installation of CompoundDb).)

## Examples

```
BiocManager::install("CompoundDb"); cdb <- CompDb(dbname = "CompDb.Hsapiens.HMDB.5.0.sqlite")
```

## Evaluation signals

- After loading with CompDb(), the database opens without error and exposes its compound and MS2 spectrum tables.
- The compound table contains exactly 217,776 rows (compound annotations).
- The MS2 spectrum table contains exactly 64,920 rows, all of them experimental — no predicted (in silico) spectra are present.
- The output is a single portable file named CompDb.Hsapiens.HMDB.5.0.sqlite with embedded source/version metadata for reproducibility.

## Limitations

- The database is tied to a specific HMDB release (version 5.0, 2 November 2021); rebuilding from a different release will change the counts of compounds and experimental MS2 spectra.
- Predicted (in silico) spectra must be excluded during the build; including them would break validation against the reported totals.
- The reference database covers Human (Hsapiens) HMDB content only and is intended as an annotation reference, not as a general-purpose HMDB API client.

## Evidence

- [abstract] Such reference databases can be created and managed with the CompoundDb package.: "Such reference databases can be created and managed with the CompoundDb package."
- [methods] CompDb database containing all compounds from HMDB version 5.0: "As a reference we load a `CompDb` database containing all compounds from the Human Metabolom Database [@wishart_hmdb_2021] version 5.0"
- [methods] dbname <- "CompDb.Hsapiens.HMDB.5.0.sqlite": "dbname <- "CompDb.Hsapiens.HMDB.5.0.sqlite""
- [readme] CompoundDb repository documentation: "Repository of the `CompoundDb` package:
  https://rformassspectrometry.github.io/CompoundDb/"
