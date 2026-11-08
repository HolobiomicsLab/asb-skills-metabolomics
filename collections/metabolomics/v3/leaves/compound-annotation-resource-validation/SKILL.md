---
name: compound-annotation-resource-validation
description: Use when constructing or auditing a reference compound/MS2-spectrum database
  (e.g., HMDB-derived CompDb.Hsapiens.HMDB.5.0.sqlite) where exact annotation counts
  must be reproducible and predicted spectra excluded.
when_to_use_negative:
- The goal is matching experimental MS2 spectra against an existing reference library
  rather than building or validating the reference database itself (use matchSpectra/MetaboAnnotation
  workflows instead).
- Predicted or in-silico spectra should be retained in the resource — this skill's
  validation explicitly requires their exclusion.
- The annotation resource is a text-based spectral library (MSP, MGF, MassBank text)
  rather than a SQL-backed CompDb database.
edam_topics:
- http://edamontology.org/topic_3366
tools:
- name: CompoundDb
  role: Creates and manages the CompDb reference database; used to build the HMDB-derived
    SQLite resource with source/version metadata and predicted spectra excluded
  repo: https://rformassspectrometry.github.io/CompoundDb/
- name: R
  role: Runtime environment; the finished database is loaded via CompDb() and its
    compound and MS2 spectrum tables inspected for row-count validation
- name: BiocManager
  role: Installation route for CompoundDb and companion RforMassSpectrometry packages
    within Bioconductor
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:648825d4e4f4a4db9b77ad2a687109860902dbee5ce89c74edb118b9cf94ef35
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - CompoundDb
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3366
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# compound-annotation-resource-validation

## Summary

Build and validate a portable compound annotation resource (CompDb SQLite database) from a versioned metabolite database release such as HMDB 5.0, excluding in-silico spectra, and verify deposited compound and experimental MS2 spectrum counts against reported totals.

## When to use

Use when constructing or auditing a reference compound/MS2-spectrum database (e.g., HMDB-derived CompDb.Hsapiens.HMDB.5.0.sqlite) where exact annotation counts must be reproducible and predicted spectra excluded.

## When NOT to use

- The goal is matching experimental MS2 spectra against an existing reference library rather than building or validating the reference database itself (use matchSpectra/MetaboAnnotation workflows instead).
- Predicted or in-silico spectra should be retained in the resource — this skill's validation explicitly requires their exclusion.
- The annotation resource is a text-based spectral library (MSP, MGF, MassBank text) rather than a SQL-backed CompDb database.

## Inputs

- HMDB version 5.0 data release (2021-11-02) compound/metabolite records and experimental MS2 spectra downloaded from hmdb.ca
- CompoundDb package (R/Bioconductor) for building the CompDb SQLite database

## Outputs

- CompDb.Hsapiens.HMDB.5.0.sqlite — portable CompDb SQLite database with 217,776 compound annotations and 64,920 experimental MS2 spectra
- Verified row counts of the compound and MS2 spectrum tables within the CompDb object

## How to apply

Download the specific source release (e.g., HMDB version 5.0, data release 2 November 2021) from hmdb.ca, keeping the release date and version as provenance metadata. Build the SQLite database with the CompoundDb package, excluding predicted (in silico) spectra so that only experimental MS2 spectra are stored, and embed source/version metadata for reproducibility. Load the finished database in R via CompDb() and inspect its compound and MS2 spectrum tables. Validate by comparing table row counts against expected totals — for the HMDB 5.0 reference build, 217,776 compound annotations and 64,920 experimental MS2 spectra — then save the file under a fixed name such as CompDb.Hsapiens.HMDB.5.0.sqlite so downstream MetaboAnnotation workflows (matchMz, matchSpectra) resolve against a verified resource.

## Related tools

- **CompoundDb** (Creates and manages the CompDb reference database; used to build the HMDB-derived SQLite resource with source/version metadata and predicted spectra excluded) — https://rformassspectrometry.github.io/CompoundDb/
- **R** (Runtime environment; the finished database is loaded via CompDb() and its compound and MS2 spectrum tables inspected for row-count validation)
- **BiocManager** (Installation route for CompoundDb and companion RforMassSpectrometry packages within Bioconductor)

## Examples

```
dbname <- "CompDb.Hsapiens.HMDB.5.0.sqlite"; cmpdb <- CompDb(dbname)
```

## Evaluation signals

- The database loads in R without error via CompDb() on the saved .sqlite file.
- Row count of the compound table equals 217,776 for the HMDB 5.0 reference build.
- Row count of the experimental MS2 spectrum table equals 64,920, with predicted (in silico) spectra confirmed absent.
- Embedded source and version metadata identifies HMDB 5.0 release dated 2021-11-02.
- The database file is saved under the reproducible name CompDb.Hsapiens.HMDB.5.0.sqlite and reused verbatim by downstream annotation steps.

## Limitations

- Counts are release-specific: totals (217,776 compounds, 64,920 experimental MS2 spectra) are valid only for the HMDB 5.0 release of 2 November 2021 and will change for other versions.
- The exclusion of predicted spectra is part of the validation definition; a database built with in-silico spectra included will fail these checks.
- Downstream use assumes R >= 4.2 (Bioconductor 3.15), per the tutorial repository installation requirements.

## Evidence

- [abstract] Such reference databases can be created and managed with the CompoundDb package.: "Such reference databases can be created and managed with the CompoundDb package."
- [methods] CompDb HMDB 5.0 reference load: "As a reference we load a `CompDb` database containing all compounds from the Human Metabolom Database [@wishart_hmdb_2021] version 5.0"
- [methods] dbname assignment: "dbname <- "CompDb.Hsapiens.HMDB.5.0.sqlite""
- [abstract] Bioconductor availability: "all packages are thoroughly unit-tested and documented and are available on GitHub and through Bioconductor."
- [readme] README CompoundDb installation: [evidence span withheld — anchor status: external_doc]
