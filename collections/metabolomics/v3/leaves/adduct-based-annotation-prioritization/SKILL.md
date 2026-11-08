---
name: adduct-based-annotation-prioritization
description: Use when a list of m/z-to-compound matches (with adduct, isotope label,
  ppm error, and source database columns) from a MetaboShiny/MetaDBparse search must
  be prioritized into putative (MSI level 2) annotations.
when_to_use_negative:
- When MSI level 1 (confirmed) annotation is required — this step only yields putative
  MSI level 2 identifications, not standard-confirmed hits.
- When the input is raw peak data — MetaboShiny explicitly does not accept raw peak
  files; matches must already come from a database search of processed m/z peak lists.
- When the compound match table lacks adduct/isotope/source-database metadata (e.g.
  from a single-database tool that records none), since filtering depends on those
  fields.
edam_operation: http://edamontology.org/operation_3754
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: MetaboShiny
  role: R/Shiny application that filters compound matches by main-peak isotope and
    adduct status, sorts by ppm error, and assigns MSI level 2 putative annotations
  repo: https://github.com/joannawolthuis/MetaboShiny
- name: MetaDBparse
  role: Companion package that builds the 30+ compound databases and performs the
    m/z database search producing the match results that this filtering step prioritizes
  repo: github.com/joannawolthuis/MetaDBparse
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1007/s11306-020-01717-8
    title: 'MetaboShiny: interactive analysis and metabolite annotation of mass spectrometry-based
      metabolomics data'
  source_content_hash: sha256:c78d277855d97ea3a333ca6d8719d41aa8e3b040cab91dab38272f1f4f543ea0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannawolthuis/MetaboShiny
  tools:
  - MetaDBparse
  - MetaboShiny
  - R
  edam_operation: http://edamontology.org/operation_3754
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# adduct-based-annotation-prioritization

## Summary

A MetaboShiny (R/Shiny) annotation step that filters m/z-to-compound matches by main-peak isotope and adduct status, then ranks survivors by ppm error to yield MSI level 2 putative compound annotations.

## When to use

Use when a list of m/z-to-compound matches (with adduct, isotope label, ppm error, and source database columns) from a MetaboShiny/MetaDBparse search must be prioritized into putative (MSI level 2) annotations.

## When NOT to use

- When MSI level 1 (confirmed) annotation is required — this step only yields putative MSI level 2 identifications, not standard-confirmed hits.
- When the input is raw peak data — MetaboShiny explicitly does not accept raw peak files; matches must already come from a database search of processed m/z peak lists.
- When the compound match table lacks adduct/isotope/source-database metadata (e.g. from a single-database tool that records none), since filtering depends on those fields.

## Inputs

- m/z peak files (MetaboAnalyst-like, MetaboShiny native or MetaboLights format; XCMS/MSnbase-derived)
- metadata table (with 'sample', 'individual' and experimental-group columns)
- built compound databases (35 supported, e.g. HMDB, FooDB, MetaboLights, user-defined/custom CSV databases)
- compound match results table with per-match adduct, isotope label, ppm error and source-database fields
- user-defined mass error margin in parts per million (ppm)

## Outputs

- filtered_annotations.csv with MSI level 2 putative-annotation labels
- prioritized per-m/z compound match table retaining adduct, isotope and database filtering fields

## How to apply

Load the compound match results produced by the MetaboShiny/MetaDBparse database search (over 30/35 supported databases, e.g. HMDB, FooDB) for the queried m/z values, each match carrying adduct, isotope label, ppm error and source-database metadata. First retain only matches on the main peak isotope (the 100% peak), as MetaboShiny does in its search results table; isotope scoring uses the M-score with a default intensity imprecision of 2%. Next filter/score the remaining matches on adduct and isotope status alongside source database, and sort survivors by ppm error within the user-defined mass tolerance (set in ppm when loading the data). Collapse rows sharing identical SMILES into single annotation entries while keeping distinct structural isomers (same formula, different SMILES) as separate rows. Assign each prioritized hit the MSI level 2 label 'putatively annotated compound' and write the table to filtered_annotations.csv. This prioritization guides users because MetaboShiny performs metabolite annotation only on significant hits from the statistical analysis, not on all raw m/z values.

## Related tools

- **MetaboShiny** (R/Shiny application that filters compound matches by main-peak isotope and adduct status, sorts by ppm error, and assigns MSI level 2 putative annotations) — https://github.com/joannawolthuis/MetaboShiny
- **MetaDBparse** (Companion package that builds the 30+ compound databases and performs the m/z database search producing the match results that this filtering step prioritizes) — github.com/joannawolthuis/MetaDBparse

## Evaluation signals

- Output table contains adduct, isotope and source-database filtering columns for every retained hit.
- All retained matches are on main-peak (100%) isotopes; non-main-peak isotope matches are absent.
- Rows are sorted by ppm error within the user-defined ppm mass tolerance.
- Rows with identical SMILES are collapsed to single entries, while same-formula compounds with different SMILES (structural isomers) remain separate rows.
- Retained hits carry MSI level 2 'putatively annotated compound' labels in filtered_annotations.csv.

## Limitations

- MetaboShiny does not automatically update databases; a database must be re-built via the 'build database' button to pick up new versions.
- Isotope scoring currently supports only the M-score, with a configurable intensity imprecision (default 2%).
- Annotation occurs only on significant hits resulting from the statistical analysis, not on all m/z values in the peak table.
- Supports over twenty commonly observed adducts and isotopes; uncommon adducts may be missing from the (user-replaceable) adduct rules table.

## Evidence

- [methods] Results were filtered first by the main peak isotopes, and subsequently sorted by ppm error: "Results were filtered first by the main peak isotopes, and subsequently sorted by ppm error"
- [methods] MetaboShiny guides users to this step by prioritizing compounds based on their isotope and adduct status: "MetaboShiny guides users to this step by prioritizing compounds based on their isotope and adduct status"
- [readme] Isotope scoring default imprecision: "Select the method to use to score compounds that have the same weight (currently only M-score available). Set the intensity imprecision (default: 2%)."
- [methods] MSI level 2 annotation: "MetaboShiny provinces MSI level 2 annotation i.e. putatively annotated compounds"
