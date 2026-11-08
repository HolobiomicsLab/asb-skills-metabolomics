---
name: sqlite-compound-database-construction
description: 'Use when a metabolomics annotation workflow needs a self-contained compound/spectral
  reference database: you have a raw public release (e.g. HMDB 5.0 XML records and
  experimental MS2 spectra) and must convert it into a CompDb SQLite file.'
when_to_use_negative:
- A ready-made CompDb database (e.g. CompDb.Hsapiens.HMDB.5.0.sqlite) already exists
  — just load it with CompDb() instead of rebuilding
- The task is matching experimental spectra against an existing reference library
  — that is the matchSpectra/matchMz functionality of MetaboAnnotation, not database
  construction
- The target compounds are outside the coverage of the source resource (e.g. non-human-metabolite
  or lipid-only references), since the build content is defined by the downloaded
  HMDB release
edam_operation: http://edamontology.org/operation_0361
tools:
- name: CompoundDb
  role: Creates and manages the CompDb SQLite reference database from the HMDB release,
    handling compound tables, MS2 spectra (excluding predicted) and source metadata
  repo: https://rformassspectrometry.github.io/CompoundDb/
- name: R
  role: Platform in which the database is built and later loaded/verified via CompDb()
- name: BiocManager
  role: Installs CompoundDb and the other ecosystem packages from Bioconductor
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:68b5d7baeae82d7ef7d33d2246c9adc8a9688d1f1b54eab32cc0a8c82fd8bd40
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - CompoundDb
  - R
  edam_operation: http://edamontology.org/operation_0361
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# sqlite-compound-database-construction

## Summary

Builds a portable CompDb SQLite database of compound annotations and experimental MS2 spectra from a public resource release (here HMDB 5.0, 2 November 2021), excluding predicted spectra, so it can be used as the reference backend for MS1/MS2-based annotation in the R Metabolomics ecosystem.

## When to use

Use when a metabolomics annotation workflow needs a self-contained compound/spectral reference database: you have a raw public release (e.g. HMDB 5.0 XML records and experimental MS2 spectra) and must convert it into a CompDb SQLite file.

## When NOT to use

- A ready-made CompDb database (e.g. CompDb.Hsapiens.HMDB.5.0.sqlite) already exists — just load it with CompDb() instead of rebuilding
- The task is matching experimental spectra against an existing reference library — that is the matchSpectra/matchMz functionality of MetaboAnnotation, not database construction
- The target compounds are outside the coverage of the source resource (e.g. non-human-metabolite or lipid-only references), since the build content is defined by the downloaded HMDB release

## Inputs

- HMDB 5.0 data release (2 November 2021) compound/metabolite records from hmdb.ca
- HMDB experimental MS2 spectra from the same release
- R session with Bioconductor >= 3.15 (R >= 4.2)

## Outputs

- CompDb SQLite database file, e.g. CompDb.Hsapiens.HMDB.5.0.sqlite
- Compound annotation table with 217,776 compounds
- Experimental (non-predicted) MS2 spectrum table with 64,920 spectra
- Embedded source/version metadata for reproducibility

## How to apply

Download the pinned public release (HMDB version 5.0, dated 2021-11-02) containing all compound/metabolite records and experimental MS2 spectra from hmdb.ca. Use the CompoundDb Bioconductor package to build the CompDb SQLite database, excluding predicted (in silico) spectra and embedding source/version metadata so the build is reproducible. Name the finished file following the ecosystem convention, e.g. CompDb.Hsapiens.HMDB.5.0.sqlite. Validate by loading it in R with CompDb() and checking that the compound table has 217,776 rows and the experimental MS2 spectrum table has 64,920 rows. This database then serves as the reference library for downstream m/z, retention-time and MS2-spectrum matching with MetaboAnnotation (e.g. matchMz, matchSpectra).

## Related tools

- **CompoundDb** (Creates and manages the CompDb SQLite reference database from the HMDB release, handling compound tables, MS2 spectra (excluding predicted) and source metadata) — https://rformassspectrometry.github.io/CompoundDb/
- **R** (Platform in which the database is built and later loaded/verified via CompDb())
- **BiocManager** (Installs CompoundDb and the other ecosystem packages from Bioconductor)

## Examples

```
BiocManager::install("CompoundDb"); cmpdb <- CompDb(dbname = "CompDb.Hsapiens.HMDB.5.0.sqlite")
```

## Evaluation signals

- Database file loads without error via CompDb() in R
- Row count of the compound table equals 217,776
- Row count of the experimental MS2 spectrum table equals 64,920, with no predicted (in silico) spectra included
- Filename follows the ecosystem convention CompDb.Hsapiens.HMDB.5.0.sqlite
- Embedded source/version metadata identifies the HMDB 5.0 release (2021-11-02) so the build is reproducible

## Limitations

- Content is bounded by the pinned source release: only HMDB 5.0 (2 November 2021) compounds and experimental spectra are included; predicted spectra are deliberately excluded
- Requires R >= 4.2 / Bioconductor 3.15 ecosystem (Spectra, CompoundDb stack) to build and consume the database
- SQLite format serves the R/MetaboAnnotation ecosystem; direct reuse in non-R annotation engines (e.g. GNPS) would need format conversion

## Evidence

- [abstract] Such reference databases can be created and managed with the CompoundDb package.: "Such reference databases can be created and managed with the CompoundDb package."
- [methods] CompDb database containing all compounds from HMDB version 5.0: "As a reference we load a `CompDb` database containing all compounds from the Human Metabolom Database [@wishart_hmdb_2021] version 5.0"
- [methods] dbname <- "CompDb.Hsapiens.HMDB.5.0.sqlite": "dbname <- "CompDb.Hsapiens.HMDB.5.0.sqlite""
- [readme] BiocManager install of CompoundDb: [evidence span withheld — anchor status: external_doc]
