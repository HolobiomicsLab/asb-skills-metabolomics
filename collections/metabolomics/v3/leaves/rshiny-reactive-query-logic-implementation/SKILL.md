---
name: rshiny-reactive-query-logic-implementation
description: Use when building an RShiny app that takes user inputs (compound classes,
  temperatures, delays, thresholds) and reactively queries an underlying stability
  database to return per-analyte recommendations, stability flags, and downloadable
  results.
when_to_use_negative:
- Protocol search is currently only available for plasma samples, so do not use the
  reactive protocol recommendation logic when the matrix is serum (serum is supported
  in other search modes)
- Do not use this skill to process or alter uploaded data in Data filtering mode;
  the app requires that uploaded data 'will not be altered or stored', it only annotates
  and flags
- Do not rely on this reactive logic for analytes absent from the underlying database
  or named outside lipid shorthand nomenclature (lipids) and RefMet nomenclature (polar
  metabolites), since analyte names are automatically matched against ALISTER's database
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3391
- http://edamontology.org/topic_0091
tools:
- name: RShiny
  role: Web-application framework in which the reactive query logic and search modes
    of ALISTER are implemented
  repo: https://github.com/Fraunhofer-ITMP/alister
- name: R
  role: Implementation language for the RShiny-based web-application and its underlying
    stability database
  repo: https://github.com/Fraunhofer-ITMP/alister
- name: ALISTER
  role: The web-app whose reactive query logic (Sample search, Analyte search, Protocol
    search, Data filtering mode) is being implemented
  repo: https://github.com/Fraunhofer-ITMP/alister
- name: RefMet Database
  role: Nomenclature source for naming polar metabolites so user inputs can be matched
    against the underlying database
  repo: https://metabolomicsworkbench.shinyapps.io/refmet_name_search/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.cca.2024.117858
    title: ALISTER – Application for lipid stability evaluation and research
  source_content_hash: sha256:6cdc37c3289d7d0dc5398cc37ee1791df85260383f3d2a5b8c87e61085d9432f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Fraunhofer-ITMP/alister
  tools:
  - ALISTER
  - R
  - RShiny
  - RefMet Database
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3391
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# RShiny reactive query logic implementation

## Summary

Implements the reactive query-and-recommendation logic of ALISTER, an RShiny web-app for pre-analytical blood sample stability in metabolomics and lipidomics, in which user selections of analyte classes, processing temperature, and delays before/after centrifugation drive search modes that output per-analyte protocol recommendations and stability flags.

## When to use

Use when building an RShiny app that takes user inputs (compound classes, temperatures, delays, thresholds) and reactively queries an underlying stability database to return per-analyte recommendations, stability flags, and downloadable results.

## When NOT to use

- Protocol search is currently only available for plasma samples, so do not use the reactive protocol recommendation logic when the matrix is serum (serum is supported in other search modes)
- Do not use this skill to process or alter uploaded data in Data filtering mode; the app requires that uploaded data 'will not be altered or stored', it only annotates and flags
- Do not rely on this reactive logic for analytes absent from the underlying database or named outside lipid shorthand nomenclature (lipids) and RefMet nomenclature (polar metabolites), since analyte names are automatically matched against ALISTER's database

## Inputs

- User-selected compound classes of interest (analytes categorized by main lipid class or RefMet main class), selectable individually or via 'Select all' groups at once
- Processing temperature inputs, with an 'Other temperature after centrifugation' checkbox for differing pre-/post-centrifugation temperatures
- Processing delays before and after centrifugation (numeric time inputs)
- Optional user-defined stability thresholds ('Define stability thresholds')
- For Data filtering mode: a .csv file with sample names in rows and analyte names in columns (tabular form), which may be empty so long as analyte names are present
- The underlying stability database of aggregated pre-analytic study data (shipped as package data in Fraunhofer-ITMP/alister)

## Outputs

- Per-analyte recommended sampling protocol table with an icon indicating expected stability under the overall recommended protocol
- Overall recommended protocol displayed above the per-analyte table
- A flow-chart visualization of the recommended sampling protocol
- A pie chart summarizing the overall influence of input conditions on the selected analyte set / expected stability percentages
- A downloadable .csv file containing the query, output, and all relevant references
- For Data filtering mode: an annotated version of the uploaded dataset with unstable analytes flagged
- A PDF file (Protocol search) depicting all possible sampling protocols in flow-charts
- A 'Variable explanation' tab for interpreting the download

## How to apply

Structure the app around a stability database aggregated from multiple pre-analytic studies, exposed through distinct reactive search modes: 'Sample search', 'Analyte search', 'Protocol search' (prospective), and 'Data filtering mode' (retrospective). In each mode, react to user selections of compound classes (categorized by main lipid class or RefMet main class), processing temperature (with an 'Other temperature after centrifugation' toggle for varying pre-/post-centrifugation temperatures), and processing delays before and after centrifugation, then match the user's input against the database, applying generalization when exact conditions are unavailable. Apply the default stability threshold of 20% fold change due to specific pre-analytical conditions, expanding it to 30% when conditions are combined (e.g. added change due to delay before and after centrifugation), with an option for the user to override these limits via 'Define stability thresholds'. In 'Protocol search' (plasma only), offer the 'Majority Vote' (default) and 'Maximize stable analytes' (strictest protocol) selection strategies, and output a recommended overall protocol above a per-analyte table, with an icon indicating whether each analyte is expected to be stable under the recommended protocol. Make every query result downloadable as a .csv file containing the query, output, and all relevant references, with a 'Variable explanation' tab to interpret the download.

## Related tools

- **RShiny** (Web-application framework in which the reactive query logic and search modes of ALISTER are implemented) — https://github.com/Fraunhofer-ITMP/alister
- **R** (Implementation language for the RShiny-based web-application and its underlying stability database) — https://github.com/Fraunhofer-ITMP/alister
- **ALISTER** (The web-app whose reactive query logic (Sample search, Analyte search, Protocol search, Data filtering mode) is being implemented) — https://github.com/Fraunhofer-ITMP/alister
- **RefMet Database** (Nomenclature source for naming polar metabolites so user inputs can be matched against the underlying database) — https://metabolomicsworkbench.shinyapps.io/refmet_name_search/

## Evaluation signals

- The .csv download must contain the original query inputs, the output, and all relevant references, and the 'Variable explanation' tab must resolve interpretation of every column
- Default threshold check: instability is flagged at 20% fold change; when conditions are combined (e.g. delay before AND after centrifugation) the threshold must expand to 30% unless overridden via 'Define stability thresholds'
- Protocol search must only appear for plasma samples; serum must route to the other search modes
- In Data filtering mode, analyte names in the uploaded .csv must be automatically matched against the database using lipid shorthand nomenclature (lipids) or RefMet nomenclature (polar metabolites), and uploaded data must remain unaltered (only annotated/flagged)
- Each analyte must receive a recommended sampling protocol plus a stable/unstable indicator icon, with the overall protocol recommended above the table

## Limitations

- Protocol search is currently only available for plasma samples
- Current app contents are limited to the influences of time delay and temperature during processing of (EDTA) plasma and serum samples
- In some cases the database will not exactly match user conditions, so generalization needs to happen (hovering over each analyte shows how the input was interpreted)
- Lipids must be named after the lipid shorthand nomenclature and polar metabolites after their RefMet nomenclature for automatic matching
- App contents are limited to pre-analytical blood sample stability in metabolomics and lipidomics; no other omics or matrices are covered

## Evidence

- [abstract] Data of multiple pre-analytic studies were aggregated into a database.: "Data of multiple pre-analytic studies were aggregated into a database."
- [methods] By default the critical threshold for assessing instability is set to 20% fold change due to specific pre-analytical conditions.: "By default the critical threshold for assessing instability is set to 20% fold change due to specific pre-analytical conditions."
- [methods] you might want to choose the strictest protocol in order to ensure stability for the maximum number of selected analytes. For this, choose 'Maximize stable analytes'.: "you might want to choose the strictest protocol in order to ensure stability for the maximum number of selected analytes. For this, choose 'Maximize stable analytes'."
- [methods] Protocol search is currently only available for plasma samples.: "Protocol search is currently only available for plasma samples."
- [readme] the app requires that the file contains the input data in tabular form, with sample names in the rows and analyte names in the columns. Your data will not be altered or stored.: "the app requires that the file contains the input data in tabular form, with sample names in the rows and analyte names in the columns. Your data will not be altered or stored."
