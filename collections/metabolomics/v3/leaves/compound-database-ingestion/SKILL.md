---
name: compound-database-ingestion
description: Reference for building a searchable m/z compound database before metabolite
  annotation in MetaboShiny — needed once per database when the user wants m/z matches
  across adducts and isotopes.
when_to_use_negative:
- When the raw MS peak list has not yet been processed — MetaboShiny does not accept
  raw peak data; peak files must come from XCMS (MetaboAnalyst export option), MSnbase
  or a similar method before any annotation step
- When the target database has already been built (checked via the 'check if database
  exists' button) and only needs loading rather than rebuilding
- When the task is downstream statistical analysis on already-annotated features —
  annotation databases are only queried on significant hits, not for every m/z in
  the peak table
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0218
tools:
- name: MetaboShiny
  role: Hosts the Database tab where compound databases are built, updated and custom
    databases are added
  repo: https://github.com/joannawolthuis/MetaboShiny
- name: MetaDBparse
  role: Standalone package containing all the compound-db specific ingestion, m/z
    generation and SQLite writing functionality
  repo: https://github.com/joannawolthuis/MetaboShiny
- name: RSQLite
  role: Backend for storing the expanded compound records so per-m/z queries execute
    in under a second
- name: enviPat
  role: Isotope and adduct calculation engine used to generate predicted m/z entries
    for each compound variant
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-020-01717-8
    title: 'MetaboShiny: interactive analysis and metabolite annotation of mass spectrometry-based
      metabolomics data'
  source_content_hash: sha256:8bb90ff42ed79b467219da46f9a5bce89764791f57e51565935aeb02fe0e58f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannawolthuis/MetaboShiny
  tools:
  - MetaDBparse
  - MetaboShiny
  - R
  - RSQLite
  - enviPat
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0218
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# compound-database-ingestion

## Summary

Building a local, searchable compound database (e.g. HMDB, FooDB or any of 35 supported sources) by downloading compound listings (SMILES, formulas, names, descriptions) and expanding each neutral mass into predicted m/z entries for over twenty adducts and isotope variants, stored in SQLite via MetaDBparse/RSQLite.

## When to use

Reference for building a searchable m/z compound database before metabolite annotation in MetaboShiny — needed once per database when the user wants m/z matches across adducts and isotopes.

## When NOT to use

- When the raw MS peak list has not yet been processed — MetaboShiny does not accept raw peak data; peak files must come from XCMS (MetaboAnalyst export option), MSnbase or a similar method before any annotation step
- When the target database has already been built (checked via the 'check if database exists' button) and only needs loading rather than rebuilding
- When the task is downstream statistical analysis on already-annotated features — annotation databases are only queried on significant hits, not for every m/z in the peak table

## Inputs

- Compound listings from one of the 35 supported databases (structures/SMILES, formulas, names, descriptions), e.g. HMDB or FooDB
- Adduct Definitions table (current adduct table shown in the 'Adducts' settings tab)
- Adduct Rules table defining how neutral masses map to ionized variants
- Optional custom database files: logo, base CSV with required columns, database name and description

## Outputs

- Local SQLite compound database containing compound records with predicted adduct/isotope m/z values, formulas and SMILES, buildable in <1 s per m/z query
- Database version number and download date displayed next to the logo after each build

## How to apply

In MetaboShiny's Database tab, first click 'check if database exists' to see whether the chosen compound database (from the 35 supported, e.g. HMDB, FooDB) has already been built; if not, press 'build database'. MetaDBparse ingests the downloaded compound listings (structures/SMILES, formulas, names, descriptions), then uses enviPat-based adduct calculations driven by the user-configurable adduct Definitions and Rules tables to expand each neutral mass into ionized variants (over twenty commonly observed adducts and isotopes). Records plus their predicted adduct/isotope m/z values, formulas and SMILES are written to a local SQLite store via RSQLite so per-m/z queries execute in under a second. Databases are not auto-updated: rebuilding is manual, and the version number and download date are recorded next to the logo for traceability. Custom in-house databases can be added via the large '+' button by uploading a logo, a base CSV with the required columns, and a database name + description, after which the new database becomes buildable following a restart.

## Related tools

- **MetaboShiny** (Hosts the Database tab where compound databases are built, updated and custom databases are added) — https://github.com/joannawolthuis/MetaboShiny
- **MetaDBparse** (Standalone package containing all the compound-db specific ingestion, m/z generation and SQLite writing functionality) — https://github.com/joannawolthuis/MetaboShiny
- **RSQLite** (Backend for storing the expanded compound records so per-m/z queries execute in under a second)
- **enviPat** (Isotope and adduct calculation engine used to generate predicted m/z entries for each compound variant)

## Examples

```
Rscript -e "MetaboShiny::start_metshi(inBrowser=T)"  # then in the Database tab: click 'check if database exists', or 'build database' for HMDB/FooDB
```

## Evaluation signals

- After clicking 'check if database exists' the button reports the database as built, with the version number and download date listed below the logo
- A per-m/z lookup in the built SQLite store returns matches in under a second, confirming the RSQLite index is populated
- The number of adduct/isotope variants generated per compound matches the over twenty commonly observed adducts configured in the Adducts Definitions and Rules tables
- For the MetaboLights lung cancer benchmark, a search including HMDB recovers creatine riboside at 264 m/z as a putative hit via the M + H adduct, confirming the database was built correctly for that source

## Limitations

- MetaboShiny does not automatically update databases — rebuilding is manual and triggered by clicking 'build database' again
- Building a database is a one-time prerequisite before any other steps; a search cannot proceed until the target database exists
- Only 35 compound databases are currently supported; anything outside that list must be added as a custom database with a logo, a correctly formatted base CSV, and a name + description, and becomes buildable only after a MetaboShiny restart

## Evidence

- [intro] MetaboShiny, through our included companion package MetaDBparse, supports over 30 databases (Table S1): "MetaboShiny, through our included companion package MetaDBparse, supports over 30 databases (Table S1)"
- [intro] supports over 30 databases and searching millions of m/z values including metabolite variants with over twenty commonly observed adducts and isotopes: "supports over 30 databases (Table S1) and is capable of searching through millions of m/z values, including metabolite variants with over twenty commonly observed adducts and isotopes"
- [methods] MetaboShiny currently supports 35 compound databases: "MetaboShiny currently supports 35 compound databases (see Table S1 for an overview of all database options)"
- [readme] The database version number and download date are listed there as well.: "To check if a database has been built, click the "check if database exists" button below the logo (Figure [Database Tab]). The database version number and download date are listed there as well."
- [methods] The MetaDBparse package containing all the search and compound-db specific functionality: [evidence span withheld — anchor status: not_found]
- [methods] a search including the HMDB uncovered creatine riboside (M + H adduct) as a putative hit: "a search including the HMDB (Fig. 1c, d) uncovered creatine riboside (M + H adduct) as a putative hit"
