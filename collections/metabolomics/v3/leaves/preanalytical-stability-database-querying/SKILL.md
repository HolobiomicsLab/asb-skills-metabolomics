---
name: preanalytical-stability-database-querying
description: Use when a set of measured lipids (lipid shorthand nomenclature) and/or
  polar metabolites (RefMet nomenclature) from (EDTA) plasma or serum needs retrospective
  stability assessment based on known sampling conditions (temperature, time delays
  before/after centrifugation) so results can be flagged.
when_to_use_negative:
- You are planning a future sampling protocol rather than assessing already-collected
  samples — use ALISTER's Protocol search instead.
- You need deep, single-analyte detail (protocol recommendation plus generalized-input
  details) for one specific metabolite/lipid — use ALISTER's Analyte search.
- You hold a full measurement table (samples in rows, analytes in columns) and want
  unstable analytes flagged and removed directly from the dataset — use ALISTER's
  Data filtering mode.
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0091
tools:
- name: ALISTER
  role: RShiny web-app whose Sample search mode performs the retrospective pre-analytical
    stability query described here
  repo: https://github.com/Fraunhofer-ITMP/alister
- name: RShiny
  role: Framework in which the flexible stability-data evaluation and the reactive
    query inputs are implemented
  repo: https://itmp.shinyapps.io/alister/
- name: RefMet Database
  role: Nomenclature source for naming polar metabolites so user analytes match the
    ALISTER database
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.cca.2024.117858
    title: ALISTER – Application for lipid stability evaluation and research
  source_content_hash: sha256:54d886b838ebb5b330710af17f5d33701fcfd4595c1d8135bc381025cdaaeb5b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Fraunhofer-ITMP/alister
  tools:
  - ALISTER
  - R
  - RShiny
  - RefMet Database
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# preanalytical-stability-database-querying

## Summary

Queries the ALISTER stability database (RShiny web-app from Fraunhofer-ITMP) to determine whether lipids and polar metabolites measured in EDTA plasma or serum can be treated as stable given user-specified processing temperature and pre-/post-centrifugation delays, applying 20%/30% fold-change instability thresholds with input generalization when exact matches are absent.

## When to use

Use when a set of measured lipids (lipid shorthand nomenclature) and/or polar metabolites (RefMet nomenclature) from (EDTA) plasma or serum needs retrospective stability assessment based on known sampling conditions (temperature, time delays before/after centrifugation) so results can be flagged or filtered before downstream analysis.

## When NOT to use

- You are planning a future sampling protocol rather than assessing already-collected samples — use ALISTER's Protocol search instead.
- You need deep, single-analyte detail (protocol recommendation plus generalized-input details) for one specific metabolite/lipid — use ALISTER's Analyte search.
- You hold a full measurement table (samples in rows, analytes in columns) and want unstable analytes flagged and removed directly from the dataset — use ALISTER's Data filtering mode.

## Inputs

- Matrix selection: plasma (EDTA) or serum
- Compound classes of interest (main lipid classes for lipids; RefMet main classes for polar metabolites)
- Processing temperature during sample handling (with optional separate pre- and post-centrifugation temperatures)
- Processing delays before centrifugation and after centrifugation
- Optional custom stability thresholds (default 20% fold change; 30% when conditions combined)

## Outputs

- Per-analyte stable/unstable classification under the specified pre-analytical conditions
- Hover explanations showing how the input was interpreted/generalized against the database
- Pie chart of stability category proportions across the selected analytes
- Downloaded .csv file containing the query, output classification, and relevant literature references

## How to apply

First select the matrix (plasma or serum), then in the ALISTER Sample search select the compound classes of interest — lipids grouped by main lipid class and polar metabolites by RefMet main class, with a 'Select all' option. Enter the processing temperature; if it varied before and after centrifugation, tick 'Other temperature after centrifugation' and supply both values. Enter processing delays before and after centrifugation separately, since some analytes respond differently to pre- versus post-centrifugation delays. Assess stability against the default critical threshold of 20% fold change for a single condition, expanded to 30% when conditions are combined (e.g. delays before plus after centrifugation); adjust these limits via 'Define stability thresholds' if a different sensitivity is needed. When the underlying database has no exact experimental match, ALISTER generalizes the input — hover each analyte to see how your input was interpreted, and click 'Citation' to view the literature the classification rests on. Read the overall effect from the pie chart of stability categories and export a .csv containing the query, per-analyte output, and all relevant references; the 'Variable explanation' tab documents every download column.

## Related tools

- **ALISTER** (RShiny web-app whose Sample search mode performs the retrospective pre-analytical stability query described here) — https://github.com/Fraunhofer-ITMP/alister
- **RShiny** (Framework in which the flexible stability-data evaluation and the reactive query inputs are implemented) — https://itmp.shinyapps.io/alister/
- **RefMet Database** (Nomenclature source for naming polar metabolites so user analytes match the ALISTER database)

## Evaluation signals

- The download .csv contains the full user query, per-analyte stability classification, and all relevant references, as described in the 'Variable explanation' tab.
- The pie chart of stability categories should sum to the number of selected analytes and show the overall influence of the input conditions.
- When the database has no exact match, the hover text for each analyte documents how the input was generalized — verify classifications are consistent with the stated approximation.
- The applied instability threshold must be 20% fold change for a single condition and 30% when conditions (e.g. delays before and after centrifugation) are combined, unless 'Define stability thresholds' was used.
- Compound class selections must follow the app's categorization: main lipid class for lipids and RefMet main class for polar metabolites.

## Limitations

- Current app contents are limited to the influences of time delay and temperature during processing of (EDTA) plasma and serum samples.
- In some cases the underlying database does not exactly match the entered conditions and generalization must occur; hover each analyte to see how the input was interpreted.
- Input analytes must follow the database nomenclature — lipid shorthand nomenclature for lipids and RefMet nomenclature for polar metabolites.
- Default thresholds (20% single condition / 30% combined) may not suit every study; explicit readjustment via 'Define stability thresholds' is required for stricter or looser criteria.

## Evidence

- [readme] Sample search is used in the following way: 1. Select the compound classes of interest from the list. Analytes are categorized by their main lipid class (when lipids) or their main class according to the RefMet database: "Sample search is used in the following way: 1. Select the compound classes of interest from the list. Analytes are categorized by their main lipid class (when lipids) or their main class according to"
- [methods] Default thresholds: "By default the critical threshold for assessing instability is set to 20% fold change due to specific pre-analytical conditions."
- [readme] Generalization of unmatched inputs: "In some cases, the database underlying ALISTER will not exactly match your conditions. In these cases, generalization needs to happen."
- [methods] Combined-condition threshold expansion: "When conditions are combined"
- [readme] Export format: "You can download a .csv-file containing your query, as well as the output and all relevant references. The tab 'Variable explanation' will answer all your questions regarding interpretation of the"
