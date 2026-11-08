---
name: annotation-hit-filtering-by-isotope-and-ppm-error
description: Use when an m/z database search (e.g. against HMDB via MetaboShiny) returns
  multiple matched compounds and you need to collapse duplicate hits, filter by isotope/adduct
  status, and rank remaining candidates by ppm error before annotation export.
when_to_use_negative:
- The input is already a filtered, validated annotation table (no new database search
  needed) — applying this skill again would only re-sort existing rows
- The goal is raw m/z matching against a database without post-match prioritization
  (the pre-matching step in MetaboShiny handles the search itself)
- The compound database is empty or not yet built — MetaboShiny requires databases
  to be built once before any search or filtering
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_3520
tools:
- name: MetaboShiny
  role: Performs the database search, then filters matches to main-peak isotopes,
    sorts by ppm error, and prioritizes compounds by isotope/adduct status before
    export
  repo: https://github.com/joannawolthuis/MetaboShiny
- name: MetaDBparse
  role: Generates the SQLite-backed compound database whose entries supply name, formula,
    SMILES, and database ID for each candidate match
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s11306-020-01717-8
    title: 'MetaboShiny: interactive analysis and metabolite annotation of mass spectrometry-based
      metabolomics data'
  source_content_hash: sha256:4dc90ac404cc2375e61c6099dc51f3795fe7305937361dd99e7f1503bf9df499
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannawolthuis/MetaboShiny
  tools:
  - MetaDBparse
  - MetaboShiny
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# annotation-hit-filtering-by-isotope-and-ppm-error

## Summary

Filters and prioritizes m/z annotation hits in MetaboShiny by retaining main-peak isotopes, then sorting matched compounds by their ppm mass error, to surface the most plausible identities for annotated metabolites.

## When to use

Use when an m/z database search (e.g. against HMDB via MetaboShiny) returns multiple matched compounds and you need to collapse duplicate hits, filter by isotope/adduct status, and rank remaining candidates by ppm error before annotation export.

## When NOT to use

- The input is already a filtered, validated annotation table (no new database search needed) — applying this skill again would only re-sort existing rows
- The goal is raw m/z matching against a database without post-match prioritization (the pre-matching step in MetaboShiny handles the search itself)
- The compound database is empty or not yet built — MetaboShiny requires databases to be built once before any search or filtering

## Inputs

- m/z values from a peaklist (e.g. 100 benchmark m/z over 60–600 range or a specific m/z such as 264)
- SQLite-backed compound database built by MetaDBparse (with m/z values for common isotopes/adducts, name, molecular formula, SMILES, database ID)
- User-set mass spectrometer error margin in ppm

## Outputs

- Filtered, sorted annotation match table with adduct and dppm columns (e.g. mz_annotation_matches.csv)
- Ranked compound matches retaining adduct and isotope-percentage fields

## How to apply

After MetaboShiny queries the SQLite-backed compound database at a user-set ppm error margin using the ppm_range helper, collapse hits sharing identical SMILES into single rows while keeping compounds with identical formulas but different structures as separate rows. Filter the matches to main-peak isotopes first, then sort by ppm error, retaining adduct and isotope-percentage fields. Prioritize compounds by their isotope and adduct status to guide the user to the most plausible hit. Export the resulting table with adduct and dppm columns (e.g. as mz_annotation_matches.csv); a correct run should recover, for example, creatine riboside as an M + H putative hit when searching 264 m/z against HMDB.

## Related tools

- **MetaboShiny** (Performs the database search, then filters matches to main-peak isotopes, sorts by ppm error, and prioritizes compounds by isotope/adduct status before export) — https://github.com/joannawolthuis/MetaboShiny
- **MetaDBparse** (Generates the SQLite-backed compound database whose entries supply name, formula, SMILES, and database ID for each candidate match)

## Evaluation signals

- Output table must retain the adduct and dppm columns after filtering and sorting
- Hits sharing identical SMILES are collapsed to one row, while compounds with the same formula but different SMILES remain separate rows
- Searching 264 m/z against HMDB should return creatine riboside as a putative M + H hit
- Rows should be ordered by ascending ppm error after sorting
- Only main-peak isotope matches remain after the isotope filter

## Limitations

- MetaboShiny does not automatically update databases; a stale database can yield outdated or missing compound hits
- The isotope scoring currently supports only the M-score method, with a default intensity imprecision of 2%
- MetaboShiny does not accept raw peak data; peaklists must first be processed with e.g. XCMS or MSnbase

## Evidence

- [methods] Results were filtered first by the main peak isotopes, and subsequently sorted by ppm error: "Results were filtered first by the main peak isotopes, and subsequently sorted by ppm error"
- [methods] MetaboShiny guides users to this step by prioritizing compounds based on their isotope and adduct status: "MetaboShiny guides users to this step by prioritizing compounds based on their isotope and adduct status"
- [methods] a search including the HMDB (Fig. 1c, d) uncovered creatine riboside (M + H adduct) as a putative hit: "a search including the HMDB (Fig. 1c, d) uncovered creatine riboside (M + H adduct) as a putative hit"
- [readme] MetaboShiny does not automatically update databases. To re-build a database of interest, click on the "build database" button: "MetaboShiny does not automatically update databases. To re-build a database of interest, click on the "build database" button"
- [readme] MetaboShiny does not accept raw peak data. We suggest using either XCMS (with the MetaboAnalyst export option) or another method of choice such as MSnbase: "MetaboShiny does not accept raw peak data. We suggest using either XCMS (with the MetaboAnalyst export option) or another method of choice such as MSnbase"
