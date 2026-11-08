---
name: mz-database-annotation
description: Use when a metabolomics analysis has produced a list of significant m/z
  values (e.g. t-test hits such as 264, 308, 441, 561 m/z) that need putative compound
  identities and MSI confidence-level assignment.
when_to_use_negative:
- The goal is pre-matching all m/z values rather than annotating significant hits
  — MetaboShiny's annotation workflow is restricted to significant hits resulting
  from the analysis (an optional pre-matching step exists for all m/z).
- The input is already a list of identified metabolites or a confirmed-identification
  table rather than unlabeled significant m/z values.
- Only MSI level 1 confirmed identifications are acceptable without further evidence
  — database matching alone yields putative (MSI level 2) annotations.
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0157
tools:
- name: MetaboShiny
  role: Interactive R/Shiny app used to match significant m/z values against the built
    database within the ppm error margin, filter by isotope/adduct status, sort by
    ppm error and assign MSI confidence levels
  repo: https://github.com/joannawolthuis/MetaboShiny
- name: MetaDBparse
  role: Companion package used to build the local compound database (e.g. HMDB), pre-generating
    m/z values for common isotopes and adducts of every compound
  repo: https://github.com/joannawolthuis/MetaDBparse
- name: R
  role: Language in which MetaboShiny/MetaDBparse are implemented and in which the
    annotation workflow (including writing annotation_counts.csv) is executed
  repo: https://github.com/joannawolthuis/MetaboShiny
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s11306-020-01717-8
    title: 'MetaboShiny: interactive analysis and metabolite annotation of mass spectrometry-based
      metabolomics data'
  source_content_hash: sha256:92eacb4731bb0a8c1394d1e7ad990a1a250cb2be7de93f34bb242242d09bead5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/joannawolthuis/MetaboShiny
  tools:
  - MetaDBparse
  - MetaboShiny
  - R
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mz-database-annotation

## Summary

Putatively annotate significant m/z hits from a metabolomics statistical analysis by matching them against locally built compound databases (e.g. HMDB via MetaDBparse) within a user-defined ppm error margin, then filter and stratify matches into Metabolomics Standards Initiative (MSI) confidence levels.

## When to use

Use when a metabolomics analysis has produced a list of significant m/z values (e.g. t-test hits such as 264, 308, 441, 561 m/z) that need putative compound identities and MSI confidence-level assignment.

## When NOT to use

- The goal is pre-matching all m/z values rather than annotating significant hits — MetaboShiny's annotation workflow is restricted to significant hits resulting from the analysis (an optional pre-matching step exists for all m/z).
- The input is already a list of identified metabolites or a confirmed-identification table rather than unlabeled significant m/z values.
- Only MSI level 1 confirmed identifications are acceptable without further evidence — database matching alone yields putative (MSI level 2) annotations.

## Inputs

- List of significant m/z hit values from statistical analysis (e.g. 264, 308, 441, 561 m/z)
- Locally built compound database (e.g. HMDB) prepared via MetaDBparse
- User-defined mass spectrometer error margin in parts per million (ppm)
- Adduct and isotope rule tables (default tables shipped with MetaboShiny or user-imported)

## Outputs

- Compound match table with compound names, formulas, SMILES, adduct/isotope status and ppm error
- annotation_counts.csv with exact row counts per MSI level (4 m/z values at MSI level 1, 264 at the next annotation level)

## How to apply

First prepare the compound database with MetaDBparse, which pre-generates m/z values for common isotopes and adducts (over twenty adducts/isotopes, 35 supported databases) of every compound; the database must be built before any other step and can be checked/rebuilt via the Database tab. Then match each significant m/z value against the database within the user-defined ppm error margin set during file import, retrieving candidate compounds such as creatine riboside as the M + H adduct of 264 m/z. Filter matches first by main peak isotopes, then sort by ppm error, collapsing compounds with identical SMILES but different names into one row. Finally assign Metabolomics Standards Initiative confidence levels, separating MSI level 1 confirmed identifications from MSI level 2 putatively annotated compounds, and emit per-level counts.

## Related tools

- **MetaboShiny** (Interactive R/Shiny app used to match significant m/z values against the built database within the ppm error margin, filter by isotope/adduct status, sort by ppm error and assign MSI confidence levels) — https://github.com/joannawolthuis/MetaboShiny
- **MetaDBparse** (Companion package used to build the local compound database (e.g. HMDB), pre-generating m/z values for common isotopes and adducts of every compound) — https://github.com/joannawolthuis/MetaDBparse
- **R** (Language in which MetaboShiny/MetaDBparse are implemented and in which the annotation workflow (including writing annotation_counts.csv) is executed) — https://github.com/joannawolthuis/MetaboShiny

## Examples

```
library(MetaboShiny); start_metshi(inBrowser=T)
```

## Evaluation signals

- annotation_counts.csv must contain exactly 4 m/z values at MSI level 1 and 264 at the next annotation level, matching the reported tallies
- 264 m/z must retrieve creatine riboside as its M + H adduct when searching HMDB
- Match list must be filtered by main peak isotopes before being sorted by ppm error
- Compounds with identical SMILES but different names must be collapsed into a single row
- All reported matches must fall within the user-defined ppm error margin set at file import

## Limitations

- DI-MS routinely produces over a hundred thousand unidentified m/z values, so annotation is restricted to significant hits to keep interpretation tractable
- MetaboShiny does not automatically update databases; they must be rebuilt manually via the 'build database' button
- Database matching provides putative (MSI level 2) annotations; confirmed MSI level 1 identifications require additional evidence beyond the database search

## Evidence

- [intro] MetaboShiny performs downstream data analysis directly on the m/z ratios and metabolite annotation occurs only on e.g. significant hits that result from the analysis.: "MetaboShiny performs downstream data analysis directly on the m/z ratios and metabolite annotation occurs only on e.g. significant hits that result from the analysis."
- [methods] Results were filtered first by the main peak isotopes, and subsequently sorted by ppm error.: "Results were filtered first by the main peak isotopes, and subsequently sorted by ppm error"
- [methods] HMDB search uncovered creatine riboside as M + H adduct of 264 m/z.: "a search including the HMDB (Fig. 1c, d) uncovered creatine riboside (M + H adduct) as a putative hit"
- [intro] supports over 30 databases and searching millions of m/z values including adducts and isotopes.: "supports over 30 databases (Table S1) and is capable of searching through millions of m/z values, including metabolite variants with over twenty commonly observed adducts and isotopes"
- [readme] MetaboShiny offers multiple metabolite databases for m/z identification and databases must be built first.: "MetaboShiny offers multiple metabolite databases for m/z identification. Before any other steps are taken, it is necessary to build the databases"
