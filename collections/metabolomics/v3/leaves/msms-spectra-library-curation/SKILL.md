---
name: msms-spectra-library-curation
description: Use when a local, self-contained reference compound/MS2 spectral database
  (e.g. SQLite CompDb) must be created from a public resource like HMDB so experimental
  fragment spectra can later be compared against a curated library.
when_to_use_negative:
- When the library already exists as an interchangeable spectral format (e.g. MGF,
  MSP, MassBank text) — import it directly into a Spectra object with MsBackendMgf/MsBackendMsp
  instead of rebuilding a CompDb.
- When the goal is running the annotation itself (m/z, retention-time or MS2 matching
  of a query dataset) rather than constructing the reference library — use MetaboAnnotation's
  matchMz/matchSpectra on an existing database.
- When predicted (in silico) MS2 spectra are required as library content — this skill's
  reference build explicitly excludes predicted spectra.
edam_operation: http://edamontology.org/operation_3759
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: CompoundDb
  role: Creates and manages the reference CompDb SQLite database from the HMDB release,
    embedding source/version metadata and excluding predicted spectra
  repo: https://rformassspectrometry.github.io/CompoundDb/
- name: Spectra
  role: Represents and handles the MS2 spectra (importable from mzML, MGF, MSP, mzXML,
    netCDF) that populate and later query the curated library
  repo: https://rformassspectrometry.github.io/Spectra/
- name: MetaboAnnotation
  role: Consumes the curated CompDb library for MS2-based annotation via matchSpectra
    against experimental fragment spectra
  repo: https://rformassspectrometry.github.io/MetaboAnnotation/
- name: BiocManager
  role: Installs CompoundDb and ecosystem dependencies from Bioconductor
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.3390/metabo12020173
    title: A Modular and Expandable Ecosystem for Metabolomics Data Annotation in
      R
  source_content_hash: sha256:0c479e67a2620c0db3d0de9d23c846b5ee64104eb6972e93a3f0484e70e06c1a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - CompoundDb
  - MetaboAnnotation
  - R
  - Spectra
  edam_operation: http://edamontology.org/operation_3759
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# msms-spectra-library-curation

## Summary

Builds and curates a portable, version-stamped reference annotation database of compounds and experimental MS2 spectra with the CompoundDb package, exemplified by the HMDB 5.0 CompDb build (CompDb.Hsapiens.HMDB.5.0.sqlite) used for downstream MS2-based metabolite annotation in the MetaboAnnotation ecosystem.

## When to use

Use when a local, self-contained reference compound/MS2 spectral database (e.g. SQLite CompDb) must be created from a public resource like HMDB so experimental fragment spectra can later be compared against a curated library.

## When NOT to use

- When the library already exists as an interchangeable spectral format (e.g. MGF, MSP, MassBank text) — import it directly into a Spectra object with MsBackendMgf/MsBackendMsp instead of rebuilding a CompDb.
- When the goal is running the annotation itself (m/z, retention-time or MS2 matching of a query dataset) rather than constructing the reference library — use MetaboAnnotation's matchMz/matchSpectra on an existing database.
- When predicted (in silico) MS2 spectra are required as library content — this skill's reference build explicitly excludes predicted spectra.

## Inputs

- HMDB 5.0 data release (2021-11-02) download from hmdb.ca: all compound/metabolite records and experimental MS2 spectra
- R >= 4.2 with Bioconductor 3.15 environment and CompoundDb installed via BiocManager

## Outputs

- CompDb.Hsapiens.HMDB.5.0.sqlite — portable CompDb SQLite database with 217,776 compound annotations and 64,920 experimental MS2 spectra (predicted spectra excluded)
- CompDb object loadable in R with embedded source/version metadata for reproducibility

## How to apply

Download a pinned release of the source database — for the reference build this is the HMDB version 5.0 data release of 2 November 2021 from hmdb.ca, containing all compound/metabolite records and experimental MS2 spectra. Build a CompDb SQLite database with CompoundDb from these files, deliberately excluding predicted (in silico) spectra so only experimental MS2 spectra enter the library, and embed source/version metadata in the database for reproducibility. Save the resulting file under a version-explicit name such as CompDb.Hsapiens.HMDB.5.0.sqlite. Load it in R via CompDb() and inspect the compound and MS2 spectrum tables; the reference build validates by containing 217,776 compound annotations and 64,920 experimental MS2 spectra. The curated CompDb then serves as the reference library in MS2-based annotation workflows, where experimental fragment spectra from mzML-derived data are compared against it via matchSpectra from MetaboAnnotation.

## Related tools

- **CompoundDb** (Creates and manages the reference CompDb SQLite database from the HMDB release, embedding source/version metadata and excluding predicted spectra) — https://rformassspectrometry.github.io/CompoundDb/
- **Spectra** (Represents and handles the MS2 spectra (importable from mzML, MGF, MSP, mzXML, netCDF) that populate and later query the curated library) — https://rformassspectrometry.github.io/Spectra/
- **MetaboAnnotation** (Consumes the curated CompDb library for MS2-based annotation via matchSpectra against experimental fragment spectra) — https://rformassspectrometry.github.io/MetaboAnnotation/
- **BiocManager** (Installs CompoundDb and ecosystem dependencies from Bioconductor)

## Examples

```
install.packages("BiocManager"); BiocManager::install("CompoundDb"); dbname <- "CompDb.Hsapiens.HMDB.5.0.sqlite"; cdb <- CompDb(dbname = dbname)
```

## Evaluation signals

- CompDb(dbname = "CompDb.Hsapiens.HMDB.5.0.sqlite") loads the database without error in R
- Row count of the compound table equals 217,776
- Row count of the MS2 spectrum table equals 64,920, confirming predicted (in silico) spectra were excluded
- Source and version metadata (HMDB 5.0, 2 November 2021 release) is embedded and retrievable from the database
- The SQLite file is portable — it can be copied and loaded by another R session without re-running the build

## Limitations

- The reference library excludes predicted spectra, so it contains only the 64,920 experimental MS2 spectra, not the full HMDB spectral content.
- The build is tied to a specific source release (HMDB 5.0 of 2021-11-02); version metadata must be embedded or reproducibility against future HMDB releases is lost.
- Requires R >= 4.2 (Bioconductor 3.15) for the ecosystem packages used to build and query the database.

## Evidence

- [abstract] Such reference databases can be created and managed with the CompoundDb package.: "Such reference databases can be created and managed with the CompoundDb package."
- [abstract] followed by an MS2-based annotation in which experimental fragment spectra are compared against a reference library: "followed by an MS2-based annotation in which experimental fragment spectra are compared against a"
- [methods] As a reference we load a `CompDb` database containing all compounds from the Human Metabolom Database [@wishart_hmdb_2021] version 5.0: "As a reference we load a `CompDb` database containing all compounds from the Human Metabolom Database [@wishart_hmdb_2021] version 5.0"
- [methods] dbname <- "CompDb.Hsapiens.HMDB.5.0.sqlite": "dbname <- "CompDb.Hsapiens.HMDB.5.0.sqlite""
- [abstract] all packages are thoroughly unit-tested and documented and are available on GitHub and through Bioconductor: "all packages are thoroughly unit-tested and documented and are available on GitHub and through Bioconductor"
- [readme] BiocManager::install("CompoundDb"): "## Support for MGF files
BiocManager::install("MsBackendMgf")

## Support for MassBank
BiocManager::install("MsBackendMassbank")"
