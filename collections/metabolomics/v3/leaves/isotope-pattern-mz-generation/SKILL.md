---
name: isotope-pattern-mz-generation
description: Use when building a compound database for m/z annotation in untargeted
  metabolomics and you need each compound's neutral mass expanded to adduct and isotope
  variants — e.g. before searching over 30 databases with MetaboShiny's MetaDBparse.
when_to_use_negative:
- MetaboShiny does not accept raw peak data — run peak picking (e.g. XCMS or MSnbase)
  first before annotation
- When you only need to re-annotate a previously significant m/z list without database-level
  isotope/adduct expansion
- When a database version must be pinned exactly and cannot be rebuilt — MetaDBparse
  requires re-building to refresh versions
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0219
tools:
- name: MetaDBparse
  role: Companion package that performs the isotope/adduct m/z generation and database
    building for over 30 compound databases
  repo: https://github.com/joannawolthuis/MetaDBparse
- name: MetaboShiny
  role: Front-end in which the database is built, version/download date is recorded,
    and the built database is queried by ppm error after annotation
  repo: https://github.com/joannawolthuis/MetaboShiny
- name: RSQLite
  role: Storage backend for the built SQLite compound database so per-m/z queries
    run quickly
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-020-01717-8
    title: 'MetaboShiny: interactive analysis and metabolite annotation of mass spectrometry-based
      metabolomics data'
  source_content_hash: sha256:9012f6093112367e5cf79b75f081ea95539b7dee8fbb3d2824a71bec819efa08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannawolthuis/MetaDBparse
  tools:
  - MetaDBparse
  - MetaboShiny
  - R
  - RSQLite
  - enviPat
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0219
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Isotope-pattern m/z generation

## Summary

Generate isotope and adduct m/z variants for every compound in a metabolite database so untargeted MS peaks can be matched to metabolite identities, using the MetaDBparse package bundled with MetaboShiny.

## When to use

Use when building a compound database for m/z annotation in untargeted metabolomics and you need each compound's neutral mass expanded to adduct and isotope variants — e.g. before searching over 30 databases with MetaboShiny's MetaDBparse.

## When NOT to use

- MetaboShiny does not accept raw peak data — run peak picking (e.g. XCMS or MSnbase) first before annotation
- When you only need to re-annotate a previously significant m/z list without database-level isotope/adduct expansion
- When a database version must be pinned exactly and cannot be rebuilt — MetaDBparse requires re-building to refresh versions

## Inputs

- Compound database listing (e.g. HMDB, FooDB) with structures/SMILES, formulas, names, descriptions
- User-configurable adduct definitions table
- Adduct rules table mapping neutral masses to ionized variants
- Desired compound database identifier from the 35 supported options

## Outputs

- Local SQLite database of compounds with predicted adduct/isotope m/z variants, formulas and SMILES
- Database version number and download date recorded for each built database

## How to apply

Build the compound database once per database of interest using MetaDBparse: ingest a compound listing (e.g. HMDB or FooDB, with structures/SMILES, formulas and names), then apply the user-configurable adduct definitions table and adduct rules table to expand each neutral mass into ionized variants using enviPat's isotope/adduct calculation. Set the mass spectrometer error margin in parts per million (ppm) before matching, and check that the database build succeeded via the 'check if database exists' button, which also records the database version number and download date for traceability. The resulting local SQLite database (via RSQLite) stores every compound together with its predicted adduct/isotope m/z variants, formulas and SMILES, enabling per-m/z queries that execute in under a second. Because MetaboShiny does not automatically update databases, rebuilding is required to pick up newer compound listings.

## Related tools

- **MetaDBparse** (Companion package that performs the isotope/adduct m/z generation and database building for over 30 compound databases) — https://github.com/joannawolthuis/MetaDBparse
- **MetaboShiny** (Front-end in which the database is built, version/download date is recorded, and the built database is queried by ppm error after annotation) — https://github.com/joannawolthuis/MetaboShiny
- **RSQLite** (Storage backend for the built SQLite compound database so per-m/z queries run quickly)

## Examples

```
docker run -p 8080:8080 -v ~/MetaboShiny/:/root/MetaboShiny/ --rm -it jcwolthuis/metaboshiny:latest Rscript -e "MetaboShiny::start_metshi(inBrowser=F)"
```

## Evaluation signals

- The 'check if database exists' button reports the database exists and shows the expected version number and download date
- A known m/z (e.g. creatine riboside at 264 m/z as the M + H adduct) is matched against HMDB within the user-set ppm error margin
- Generated m/z list includes over twenty commonly observed adducts and isotope variants per compound
- Per-m/z queries against the SQLite database execute in under a second
- Rebuilding a database updates the recorded version number and download date

## Limitations

- MetaboShiny does not automatically update databases; rebuilding is manual
- MetaboShiny does not accept raw peak data — annotation only makes sense on picked peaklists
- The isotope scoring method currently available is only M-score, with a default intensity imprecision of 2%
- Only the databases the user has explicitly built are available for annotation

## Evidence

- [intro] MetaboShiny, through our included companion package MetaDBparse, supports over 30 databases: "MetaboShiny, through our included companion package MetaDBparse, supports over 30 databases (Table S1) and is capable of searching through millions of m/z values"
- [intro] including metabolite variants with over twenty commonly observed adducts and isotopes: "including metabolite variants with over twenty commonly observed adducts and isotopes"
- [methods] MetaboShiny currently supports 35 compound databases: "MetaboShiny currently supports 35 compound databases (see Table S1 for an overview of all database options)"
- [readme] Before any other steps are taken, it is necessary to build the databases that the user is interested in: "Before any other steps are taken, it is necessary to build the databases that the user is interested in. Each database only needs to be built once."
- [readme] The database version number and download date are listed there as well: "The database version number and download date are listed there as well."
- [readme] MetaboShiny does not automatically update databases: "MetaboShiny does not automatically update databases. To re-build a database of interest, click on the "build database" button below the logo in the Database Tab."
- [methods] Results were filtered first by the main peak isotopes, and subsequently sorted by ppm error: "Results were filtered first by the main peak isotopes, and subsequently sorted by ppm error"
