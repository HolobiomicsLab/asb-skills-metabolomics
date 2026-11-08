---
name: adduct-mz-variant-prediction
description: Use when a neutral-mass compound database (e.g. HMDB, FooDB) must be
  converted into an m/z-searchable annotation resource, or when matching observed
  peak m/z values (with a known ppm error margin) against compounds whose adduct and
  isotope variants are not yet enumerated.
when_to_use_negative:
- The input is a raw peak file — MetaboShiny does not accept raw peak data; use XCMS
  (MetaboAnalyst export option) or MSnbase to produce a peaklist first.
- Only a single fixed database is required and no adduct/isotope variant enumeration
  is needed — a fixed-version single-database annotator may suffice.
- The goal is formula prediction rather than compound matching — that uses the separate
  formula prediction and lookup settings.
tools:
- name: MetaboShiny
  role: R/RShiny app that hosts the database-building panel and the m/z search with
    isotope/adduct prioritization
  repo: https://github.com/joannawolthuis/MetaboShiny
- name: MetaDBparse
  role: Companion package that ingests over 30 databases and generates the searchable
    m/z entries including metabolite variants
  repo: https://github.com/joannawolthuis/MetaDBparse
- name: enviPat
  role: Isotope/adduct calculation engine used to generate predicted m/z entries
- name: RSQLite
  role: Backend that stores the built compound database for sub-second per-m/z queries
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1007/s11306-020-01717-8
    title: 'MetaboShiny: interactive analysis and metabolite annotation of mass spectrometry-based
      metabolomics data'
  source_content_hash: sha256:8a8ac9187b9c59d6cd180438f8bc4a1f23370d50337c06b58fdf36fe60b556fe
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# adduct-mz-variant-prediction

## Summary

Generate searchable ionized m/z variants for every compound in a metabolite database by combining over twenty commonly observed adducts and isotope states, so that putative annotation of untargeted LC-MS/DI-MS peaklists can span many databases instead of a single one.

## When to use

Use when a neutral-mass compound database (e.g. HMDB, FooDB) must be converted into an m/z-searchable annotation resource, or when matching observed peak m/z values (with a known ppm error margin) against compounds whose adduct and isotope variants are not yet enumerated.

## When NOT to use

- The input is a raw peak file — MetaboShiny does not accept raw peak data; use XCMS (MetaboAnalyst export option) or MSnbase to produce a peaklist first.
- Only a single fixed database is required and no adduct/isotope variant enumeration is needed — a fixed-version single-database annotator may suffice.
- The goal is formula prediction rather than compound matching — that uses the separate formula prediction and lookup settings.

## Inputs

- Compound database listing (e.g. HMDB, FooDB) with structures/SMILES, formulas, names and descriptions
- User-configurable adduct definitions table
- Adduct rules table (defines how neutral mass maps to ionized variants)
- Observed peaklist m/z values with the mass spectrometer error margin set in ppm
- Optional custom base CSV file (logo, columns shown in the '+' pop-up, database name + description) for user-defined databases

## Outputs

- Local SQLite compound database (via RSQLite) containing compound records with predicted adduct/isotope m/z variants, formulas and SMILES
- Per-m/z compound match list sorted by ppm error and filtered by isotope/adduct status
- Database version number and download date recorded with the build

## How to apply

Start from a compound listing downloaded into MetaDBparse (structures/SMILES, formulas, names, descriptions) plus user-configurable adduct 'Definitions' and adduct 'Rules' tables. For each compound, compute predicted m/z entries across more than twenty commonly observed adducts and isotope states using the enviPat-based calculation, then write the records with their variants, formulas and SMILES into a local SQLite database (RSQLite) so per-m/z queries execute in under a second. When matching, prioritize compounds by isotope and adduct status — filter first by the main peak isotopes, then sort by ppm error against the user-set mass-spectrometer error margin (in parts per million). The database version number and download date are recorded so the build is traceable; MetaboShiny does not auto-update databases, so re-build explicitly via the 'build database' button when needed.

## Related tools

- **MetaboShiny** (R/RShiny app that hosts the database-building panel and the m/z search with isotope/adduct prioritization) — https://github.com/joannawolthuis/MetaboShiny
- **MetaDBparse** (Companion package that ingests over 30 databases and generates the searchable m/z entries including metabolite variants) — https://github.com/joannawolthuis/MetaDBparse
- **enviPat** (Isotope/adduct calculation engine used to generate predicted m/z entries)
- **RSQLite** (Backend that stores the built compound database for sub-second per-m/z queries)

## Examples

```
Rscript -e "MetaboShiny::start_metshi(inBrowser=F)"  # then Database tab: build database → Search tab: set ppm margin and match m/z values
```

## Evaluation signals

- The built database reports a version number and download date when 'check if database exists' is clicked.
- Per-m/z lookup against the built SQLite database returns matches in under a second.
- Match list is filtered by main-peak isotopes first and then sorted by ppm error, not by name or intensity.
- Compounds sharing the same mass are disambiguated by isotope/adduct-based scoring (e.g. M-score with default 2% intensity imprecision).
- Number of searchable entries exceeds the compound count because variants with over twenty adducts/isotopes were added.

## Limitations

- MetaboShiny does not automatically update databases — re-build via the 'build database' button to refresh.
- Isotope scoring currently only offers the M-score method, with the intensity imprecision defaulting to 2%.
- Adduct definitions and rules are user-configurable tables; a wrong rule table silently produces incorrect variants.
- Untargeted analysis produces annotation at MSI level 2 (putatively annotated compounds), not confirmed identities without additional evidence.

## Evidence

- [intro] Over thirty databases and variant enumeration: "MetaboShiny, through our included companion package MetaDBparse, supports over 30 databases (Table S1) and is capable of searching through millions of m/z values"
- [intro] Adduct and isotope coverage: "including metabolite variants with over twenty commonly observed adducts and isotopes"
- [methods] Prioritization and sorting of matches: "Results were filtered first by the main peak isotopes, and subsequently sorted by ppm error"
- [methods] Isotope/adduct priority guidance: "MetaboShiny guides users to this step by prioritizing compounds based on their isotope and adduct status"
- [readme] Database must be built before use: "Before any other steps are taken, it is necessary to build the databases that the user is interested in. Each database only needs to be built once."
- [readme] No automatic database update: "MetaboShiny does not automatically update databases. To re-build a database of interest, click on the 'build database' button"
