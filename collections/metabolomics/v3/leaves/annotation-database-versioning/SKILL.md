---
name: annotation-database-versioning
description: Use when building or reproducing a reference annotation database (e.g.
  from HMDB, MassBank) so that downstream MetaboAnnotation matching runs against a
  fixed, documented compound and MS2 spectrum version rather than a shifting external
  resource.
when_to_use_negative:
- The goal is annotation matching itself (m/z, RT or MS2 comparisons) — use MetaboAnnotation
  matchMz/matchSpectra against an already-built CompDb instead of rebuilding one
- Only in silico/predicted MS2 spectra are needed — this skill explicitly excludes
  predicted spectra from the build
- The reference data are already distributed as a ready-to-load CompDb SQLite file
  for the exact release needed
edam_topics:
- http://edamontology.org/topic_3383
tools:
- name: CompoundDb
  role: Creates and manages the reference annotation database; builds the CompDb SQLite
    file from a source release with provenance metadata and excludes predicted spectra
  repo: https://rformassspectrometry.github.io/CompoundDb/
- name: R
  role: Language in which the CompDb build and downstream loading/verification (CompDb(),
    table row counts) are performed
- name: BiocManager
  role: Used to install CompoundDb and the rest of the Bioconductor annotation ecosystem
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:64bcadfacf1fb6c51da86a0a8a50b4a9a8e89feba806a5f2b9b2021623437e5e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - CompoundDb
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3383
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# annotation-database-versioning

## Summary

Create and manage versioned, portable compound annotation reference databases with the CompoundDb package, embedding source data release (e.g. HMDB 5.0 of 2 November 2021), version and provenance metadata into a self-contained SQLite CompDb file used downstream for MS1/MS2-based annotation in the R metabolomics ecosystem.

## When to use

Use when building or reproducing a reference annotation database (e.g. from HMDB, MassBank) so that downstream MetaboAnnotation matching runs against a fixed, documented compound and MS2 spectrum version rather than a shifting external resource.

## When NOT to use

- The goal is annotation matching itself (m/z, RT or MS2 comparisons) — use MetaboAnnotation matchMz/matchSpectra against an already-built CompDb instead of rebuilding one
- Only in silico/predicted MS2 spectra are needed — this skill explicitly excludes predicted spectra from the build
- The reference data are already distributed as a ready-to-load CompDb SQLite file for the exact release needed

## Inputs

- Reference database source release (e.g. HMDB version 5.0 compound/metabolite records and experimental MS2 spectra from hmdb.ca)
- CompoundDb package configuration for the build (metadata: source, version, date)

## Outputs

- Portable CompDb SQLite database file, e.g. CompDb.Hsapiens.HMDB.5.0.sqlite, with compound annotations and experimental MS2 spectra tables
- Versioned database object loadable via CompDb() for use as a reference library in matchSpectra/matchMz workflows

## How to apply

Download the chosen reference release (e.g. HMDB version 5.0 data release of 2021-11-02 with all compound/metabolite records and experimental MS2 spectra from hmdb.ca), then build the CompDb SQLite database with CompoundDb while excluding predicted (in silico) spectra so only experimental MS2 spectra are retained. Embed source and version metadata in the database so the build is reproducible and the resource can be cited unambiguously. Load the finished file in R via CompDb(), inspect its compound and MS2 spectrum tables, and verify row counts against expected totals before naming the file with the version baked in (e.g. CompDb.Hsapiens.HMDB.5.0.sqlite). The resulting CompDb then serves as the reference library for m/z- and MS2-based matching with MetaboAnnotation.

## Related tools

- **CompoundDb** (Creates and manages the reference annotation database; builds the CompDb SQLite file from a source release with provenance metadata and excludes predicted spectra) — https://rformassspectrometry.github.io/CompoundDb/
- **R** (Language in which the CompDb build and downstream loading/verification (CompDb(), table row counts) are performed)
- **BiocManager** (Used to install CompoundDb and the rest of the Bioconductor annotation ecosystem)

## Examples

```
library(CompoundDb); cdb <- CompDb(dbname = "CompDb.Hsapiens.HMDB.5.0.sqlite")
```

## Evaluation signals

- Database loads successfully in R via CompDb() without errors
- Compound table row count equals the expected total for the release (217,776 compounds for HMDB 5.0 of 2 November 2021)
- MS2 spectrum table row count equals the expected total of experimental (non-predicted) spectra (64,920 for HMDB 5.0)
- Embedded metadata records the correct source, version and release date (HMDB 5.0, 2021-11-02)
- Database file is portable/self-contained and named to encode the version (CompDb.Hsapiens.HMDB.5.0.sqlite)

## Limitations

- Spectra flagged as predicted/in silico in the source release are excluded, so the database contains only experimental MS2 spectra and cannot support purely predicted-spectrum matching
- The database is tied to a specific source release; results downstream are only reproducible against the same versioned CompDb file
- No changelog was found in the source material, so tracking changes between releases must be handled externally

## Evidence

- [abstract] Reference databases built from HMDB 5.0 with CompoundDb: "Such reference databases can be created and managed with the CompoundDb package."
- [methods] CompDb reference loaded from HMDB 5.0: "As a reference we load a `CompDb` database containing all compounds from the Human Metabolom Database [@wishart_hmdb_2021] version 5.0"
- [methods] Versioned CompDb SQLite file name: "dbname <- "CompDb.Hsapiens.HMDB.5.0.sqlite""
- [readme] Installing CompoundDb from the tutorial repository instructions: "BiocManager::install("MetaboAnnotation")
BiocManager::install("CompoundDb")"
