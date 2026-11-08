---
name: duplicate-smiles-collapsing
description: Use when a compound-database m/z match table contains multiple database
  entries describing the same chemical structure (identical SMILES) and you need one
  deduplicated putative annotation per structure without discarding true structural
  isomers.
when_to_use_negative:
- Matches have no structural identifiers (SMILES) to compare, so identical structures
  cannot be recognized and collapsing would be arbitrary
- The goal is to merge all hits with the same molecular formula — this skill must
  keep same-formula/different-SMILES structural isomers as separate rows
- You are at the raw m/z database search stage (MetaDBparse database building or m/z
  pre-matching) before isotope/adduct filtering has been applied
edam_operation: http://edamontology.org/operation_3632
edam_topics:
- http://edamontology.org/topic_3334
- http://edamontology.org/topic_0092
tools:
- name: MetaboShiny
  role: 'R/Shiny application that performs the annotation step: filters matches by
    main-peak isotope and adduct status, sorts by ppm error, collapses identical-SMILES
    rows and assigns MSI level 2 labels'
  repo: https://github.com/joannawolthuis/MetaboShiny
- name: MetaDBparse
  role: Companion package that generates the compound database search results (with
    over 20 adducts/isotopes across 35+ databases) whose duplicate-SMILES rows are
    collapsed
  repo: https://github.com/joannawolthuis/MetaDBparse
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-020-01717-8
    title: 'MetaboShiny: interactive analysis and metabolite annotation of mass spectrometry-based
      metabolomics data'
  source_content_hash: sha256:5c2dbc25265e2b15308dae2d6f300f17a0ff8c10af1ee7b4f6ec967c27d1d781
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannawolthuis/MetaboShiny
  tools:
  - MetaDBparse
  - MetaboShiny
  - R
  edam_operation: http://edamontology.org/operation_3632
  edam_topics:
  - http://edamontology.org/topic_3334
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# duplicate-smiles-collapsing

## Summary

Within MetaboShiny's annotation workflow, compound matches that survive isotope/adduct/database filtering and ppm-error sorting are collapsed into single annotation entries when they share an identical SMILES, while structural isomers (same formula, different SMILES) are retained as separate rows before MSI level 2 'putatively annotated compound' labels are assigned.

## When to use

Use when a compound-database m/z match table contains multiple database entries describing the same chemical structure (identical SMILES) and you need one deduplicated putative annotation per structure without discarding true structural isomers.

## When NOT to use

- Matches have no structural identifiers (SMILES) to compare, so identical structures cannot be recognized and collapsing would be arbitrary
- The goal is to merge all hits with the same molecular formula — this skill must keep same-formula/different-SMILES structural isomers as separate rows
- You are at the raw m/z database search stage (MetaDBparse database building or m/z pre-matching) before isotope/adduct filtering has been applied

## Inputs

- Compound match results from a MetaboShiny/MetaDBparse database search for queried m/z values, with per-match adduct, isotope label, ppm error and source-database fields
- User-defined mass spectrometer error margin in parts per million (ppm) set during project/file import

## Outputs

- filtered_annotations.csv containing deduplicated, prioritized compound annotations with adduct, isotope and database filtering fields
- MSI level 2 'putatively annotated compound' labels for each retained hit

## How to apply

Start from the compound match results produced by the MetaboShiny/MetaDBparse database search for each queried m/z, keeping per-match metadata (adduct, isotope label, ppm error, source database). First filter matches to main peak isotopes (100% peak) and score/filter on isotope and adduct status alongside source database, sorting survivors by ppm error within the user-defined mass spectrometer ppm tolerance set at project load time. Then group remaining rows by SMILES and collapse rows sharing an identical SMILES into a single annotation entry; deliberately keep rows with the same molecular formula but different SMILES, since MetaboShiny treats structural isomers as distinct putative hits. Assign each surviving entry an MSI level 2 'putatively annotated compound' label and write the prioritized table to filtered_annotations.csv. The rationale is that identical structures matched via different databases or adduct listings represent one metabolite identity, whereas differing SMILES represent separable isomers the user must adjudicate.

## Related tools

- **MetaboShiny** (R/Shiny application that performs the annotation step: filters matches by main-peak isotope and adduct status, sorts by ppm error, collapses identical-SMILES rows and assigns MSI level 2 labels) — https://github.com/joannawolthuis/MetaboShiny
- **MetaDBparse** (Companion package that generates the compound database search results (with over 20 adducts/isotopes across 35+ databases) whose duplicate-SMILES rows are collapsed) — https://github.com/joannawolthuis/MetaDBparse

## Evaluation signals

- filtered_annotations.csv contains adduct, isotope and database filtering columns as defined in the workflow
- No two retained rows share an identical SMILES string, while same-formula rows with different SMILES remain separate entries
- Every retained hit carries an MSI level 2 'putatively annotated compound' label
- Retained hits are sorted by ppm error and all fall within the user-defined ppm tolerance configured at file import
- All retained matches originate from main peak (100%) isotope entries

## Limitations

- Databases are not automatically updated by MetaboShiny, so collapsing results depend on potentially stale database versions that must be manually re-built
- Only the M-score method is available for scoring compounds with the same weight, and isotope scoring uses a default intensity imprecision of 2%, which constrains how matches are ranked before collapsing
- Prioritized annotations remain MSI level 2 (putative); confirmed identity requires separate validation

## Evidence

- [methods] filter-then-collapse ordering: "Results were filtered first by the main peak isotopes, and subsequently sorted by ppm error"
- [methods] prioritization rationale: "MetaboShiny guides users to this step by prioritizing compounds based on their isotope and adduct status"
- [methods] MSI level 2 output labels: "MetaboShiny provinces MSI level 2 annotation i.e. putatively annotated compounds"
- [readme] isotope scoring parameters: "Select the method to use to score compounds that have the same weight (currently only M-score available). Set the intensity imprecision (default: 2%)."
- [intro] databases behind the matches: "supports over 30 databases (Table S1) and is capable of searching through millions of m/z values, including metabolite variants with over twenty commonly observed adducts and isotopes"
