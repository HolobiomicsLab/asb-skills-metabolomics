---
name: putative-metabolite-annotation-msi-level-2
description: Use when untargeted LC-MS or DI-MS metabolomics data (peak lists with
  m/z values in ppm tolerance) require putative metabolite identities on significant
  hits rather than raw peak interpretation, and multiple compound databases (35 supported)
  should be searched with adduct/isotope-aware matching.
when_to_use_negative:
- Input is already a feature table with confirmed metabolite identities (MSI level
  1) — no database matching is needed.
- The analysis goal is raw peak detection from unprocessed spectra — MetaboShiny does
  not accept raw peak data; peak lists must first be produced by XCMS (with MetaboAnalyst
  export), MSnbase or a similar method.
- The user requires structural confirmation by tandem-MS spectral matching rather
  than accurate-mass-only putative annotation; this skill yields level 2 identities
  only.
edam_operation: http://edamontology.org/operation_3648
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0121
tools:
- name: MetaboShiny
  role: Interactive Shiny application that runs the compound-database search, computes
    ppm windows, filters/sorts matches by adduct and isotope, and exports the annotation
    table.
  repo: https://github.com/joannawolthuis/MetaboShiny
- name: MetaDBparse
  role: Companion package that builds and updates the local compound databases (over
    30 sources, e.g. HMDB) containing m/z values for common adducts and isotopes,
    used as the search target.
  repo: https://github.com/joannawolthuis/MetaDBparse
- name: R
  role: Programming language in which both MetaboShiny and MetaDBparse are implemented
    and in which the annotation pipeline is executed.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-020-01717-8
    title: 'MetaboShiny: interactive analysis and metabolite annotation of mass spectrometry-based
      metabolomics data'
  source_content_hash: sha256:976fefbec3abac24257bdae98ae93112919f16db07501e67b6b4d2f623babe53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannawolthuis/MetaboShiny
  tools:
  - MetaDBparse
  - MetaboShiny
  - R
  edam_operation: http://edamontology.org/operation_3648
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Putative metabolite annotation (MSI level 2)

## Summary

Performs database-driven m/z matching against locally built compound databases (via MetaDBparse) to assign putative metabolite identities and adduct information to significant m/z peaks, yielding MSI level 2 ('putatively annotated compounds') annotations.

## When to use

Use when untargeted LC-MS or DI-MS metabolomics data (peak lists with m/z values in ppm tolerance) require putative metabolite identities on significant hits rather than raw peak interpretation, and multiple compound databases (35 supported) should be searched with adduct/isotope-aware matching.

## When NOT to use

- Input is already a feature table with confirmed metabolite identities (MSI level 1) — no database matching is needed.
- The analysis goal is raw peak detection from unprocessed spectra — MetaboShiny does not accept raw peak data; peak lists must first be produced by XCMS (with MetaboAnalyst export), MSnbase or a similar method.
- The user requires structural confirmation by tandem-MS spectral matching rather than accurate-mass-only putative annotation; this skill yields level 2 identities only.

## Inputs

- m/z peak list table (MetaboShiny-native, MetaboAnalyst-like, or MetaboLights format; positive and negative mode peak files) with per-sample intensities
- Prebuilt compound database (SQLite, generated locally via MetaDBparse from one of 35 supported sources such as HMDB or FooDB)
- Mass-spectrometer error margin in parts per million (ppm)
- Adduct and isotope rules tables (e.g. M + H, M + NH4, M + Na definitions)
- List of significant m/z hits from prior statistical analysis (e.g. t-test, ANOVA, volcano plot)

## Outputs

- Annotation table (mz_annotation_matches.csv) with matched compound name, molecular formula, SMILES, database ID, adduct, isotope percentage and dppm (ppm error) columns
- Per-m/z compound match list filtered by main-peak isotopes and sorted by ppm error
- MSI level 2 ('putatively annotated compounds') identities for each significant m/z

## How to apply

Build the compound database(s) of interest once locally through the companion MetaDBparse package, which stores m/z values for common isotopes and adducts (e.g. M + H) of every compound in an SQLite-backed store with name, molecular formula, SMILES and database ID. Load the query m/z peak list (e.g. the positive and negative mode MetaboShiny-native or MetaboLights peak tables from the paper's benchmark pool of 100 m/z values spanning 60–600 m/z) together with the user-set mass-spectrometer error margin in ppm; the ppm_range helper computes each accepted mass window. For every query m/z, retrieve database entries whose adduct/isotope-derived m/z falls inside that window, collapse hits sharing identical SMILES into single rows while retaining compounds with identical formulas but different structures/SMILES as separate rows, then filter matches to main-peak isotopes first and sort by ppm error, keeping the adduct and isotope-percentage fields. Export the result as an annotation table with adduct and dppm columns (mz_annotation_matches.csv). Annotate only significant hits resulting from prior statistical analysis (e.g. the four m/z values 264, 308, 441, 561 predictive for cancer status in the Mathé et al. lung cancer dataset), where a search against HMDB returned creatine riboside (M + H adduct) as a putative MSI level 2 hit.

## Related tools

- **MetaboShiny** (Interactive Shiny application that runs the compound-database search, computes ppm windows, filters/sorts matches by adduct and isotope, and exports the annotation table.) — https://github.com/joannawolthuis/MetaboShiny
- **MetaDBparse** (Companion package that builds and updates the local compound databases (over 30 sources, e.g. HMDB) containing m/z values for common adducts and isotopes, used as the search target.) — https://github.com/joannawolthuis/MetaDBparse
- **R** (Programming language in which both MetaboShiny and MetaDBparse are implemented and in which the annotation pipeline is executed.)

## Examples

```
docker run -p 8080:8080 -v ~/MetaboShiny/:/root/MetaboShiny/:cached --rm -it jcwolthuis/metaboshiny:latest Rscript -e "MetaboShiny::start_metshi(inBrowser=F)"
```

## Evaluation signals

- A benchmark search of 264 m/z against HMDB returns creatine riboside with M + H adduct as a putative hit (expected result from the paper).
- The exported mz_annotation_matches.csv contains adduct and dppm columns for every row and no rows with a blank adduct assignment for main-peak isotopes.
- Hits sharing identical SMILES are collapsed into a single row; compounds with identical molecular formula but different SMILES remain as separate rows.
- Rows are restricted to main-peak isotopes first and subsequently sorted by ppm error (dppm ascending).
- Only m/z values inside the user-set ppm window (ppm_range) appear as matches — no hits fall outside the accepted mass margin.

## Limitations

- Databases are not automatically updated; each database must be rebuilt via the 'build database' button when a new version is desired.
- MetaboShiny does not accept raw peak data — peak lists must be pre-generated (e.g. via XCMS or MSnbase).
- Annotation is putative (MSI level 2) only; no structural confirmation beyond accurate mass, adduct and formula/SMILES information is provided.
- Custom/in-house databases require manual upload of a logo, a base csv file with specified columns, and a name + description; they become available only after restart.

## Evidence

- [methods] MetaboShiny provinces MSI level 2 annotation i.e. putatively annotated compounds: "MetaboShiny provinces MSI level 2 annotation i.e. putatively annotated compounds"
- [methods] a search including the HMDB (Fig. 1c, d) uncovered creatine riboside (M + H adduct) as a putative hit: "a search including the HMDB (Fig. 1c, d) uncovered creatine riboside (M + H adduct) as a putative hit"
- [methods] Results were filtered first by the main peak isotopes, and subsequently sorted by ppm error: "Results were filtered first by the main peak isotopes, and subsequently sorted by ppm error"
- [intro] MetaboShiny, through our included companion package MetaDBparse, supports over 30 databases (Table S1) and is capable of searching through millions of m/z values: "MetaboShiny, through our included companion package MetaDBparse, supports over 30 databases (Table S1) and is capable of searching through millions of m/z values"
- [readme] MetaboShiny does not accept raw peak data. We suggest using either XCMS (with the MetaboAnalyst export option) or another method of choice such as MSnbase.: "MetaboShiny does not accept raw peak data. We suggest using either XCMS (with the MetaboAnalyst export option) or another method of choice such as MSnbase."
- [methods] This work identified 4 m/z values at MSI level 1, 264 m/z, 308 m/z, 441 m/z and 561 m/z predictive for cancer status: [evidence span withheld — anchor status: not_found]
