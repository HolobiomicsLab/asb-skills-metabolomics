---
name: blood-matrix-selection-plasma-serum
description: 'Use when modeling or reproducing ALISTER''s app routing: a matrix choice
  (plasma/serum) must gate which search modes (Protocol, Sample, Analyte, Data filtering)
  a prospective or retrospective user can access.'
when_to_use_negative:
- The analysis concerns stability thresholds (e.g. the default 20% fold-change threshold
  expanded to 30% when conditions are combined) rather than which search modes are
  reachable
- The user has already entered a specific search mode (Sample search, Analyte search,
  Protocol search, Data filtering mode) and needs query-level instructions instead
  of matrix-to-mode routing
- Data filtering of a user-supplied .csv (sample names in rows, analyte names in columns)
  is the actual goal, not the app's routing logic
edam_topics: []
tools:
- name: R
  role: Language used to encode the routing(matrix, scenario) dispatch function and
    produce routing_map.csv
  repo: https://github.com/Fraunhofer-ITMP/alister
- name: RShiny
  role: Framework of the ALISTER web-application whose matrix-selection and search-mode
    flow this skill reproduces
  repo: https://github.com/Fraunhofer-ITMP/alister
- name: Shiny
  role: Deployment framework hosting the ALISTER app at itmp.shinyapps.io/alister
    whose routing is being modeled
  repo: https://github.com/Fraunhofer-ITMP/alister
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.cca.2024.117858
    title: ALISTER – Application for lipid stability evaluation and research
  source_content_hash: sha256:77ae81d443cb2298df3fedb47fac1eec59acd370aa0f56e218fdbbab71caa96c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Fraunhofer-ITMP/alister
  tools:
  - R
  - RShiny
  - Shiny
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# blood-matrix-selection-plasma-serum

## Summary

Encodes ALISTER's first-step blood matrix selection (plasma vs serum) and the downstream dispatch of search modes, enforcing that Protocol search is restricted to plasma while retrospective modes (Sample search, Analyte search, Data filtering mode) and Analyte search remain available.

## When to use

Use when modeling or reproducing ALISTER's app routing: a matrix choice (plasma/serum) must gate which search modes (Protocol, Sample, Analyte, Data filtering) a prospective or retrospective user can access.

## When NOT to use

- The analysis concerns stability thresholds (e.g. the default 20% fold-change threshold expanded to 30% when conditions are combined) rather than which search modes are reachable
- The user has already entered a specific search mode (Sample search, Analyte search, Protocol search, Data filtering mode) and needs query-level instructions instead of matrix-to-mode routing
- Data filtering of a user-supplied .csv (sample names in rows, analyte names in columns) is the actual goal, not the app's routing logic

## Inputs

- matrix selection string ('plasma' or 'serum') as chosen in ALISTER's first app step
- scenario string ('prospective' or 'retrospective') describing the pre-analytical question type
- 2x2 evaluation grid of matrix x scenario combinations

## Outputs

- routing_map.csv decision table with columns matrix, scenario, available_modes (one row per matrix-scenario combination)
- mode-generalization fallback notes for inputs not exactly matched by the underlying database

## How to apply

Implement an R dispatcher routing(matrix, scenario) over the 2x2 grid of matrix (plasma/serum) by scenario (prospective/retrospective), returning the admissible ALISTER search modes per README: retrospective scenarios map to Sample search, Analyte search and Data filtering mode; prospective scenarios map to Protocol search and Analyte search; the plasma-only constraint on Protocol search is hard-coded because 'Protocol search is currently only available for plasma samples', so serum rows must drop it. Treat 'Majority Vote' (default) vs 'Maximize stable analytes' as downstream Protocol-search options, not routing decisions. During evaluation, record mode-generalization fallbacks for cases where user conditions cannot be exactly matched by the underlying database. Write the resulting decision table (columns: matrix, scenario, available_modes) to routing_map.csv and validate it: one row per matrix-scenario combination, at least one available mode each, Protocol search appearing exclusively in plasma rows, and Analyte search appearing under both scenarios.

## Related tools

- **R** (Language used to encode the routing(matrix, scenario) dispatch function and produce routing_map.csv) — https://github.com/Fraunhofer-ITMP/alister
- **RShiny** (Framework of the ALISTER web-application whose matrix-selection and search-mode flow this skill reproduces) — https://github.com/Fraunhofer-ITMP/alister
- **Shiny** (Deployment framework hosting the ALISTER app at itmp.shinyapps.io/alister whose routing is being modeled) — https://github.com/Fraunhofer-ITMP/alister

## Examples

```
routing("plasma", "prospective")  # returns c("Protocol search", "Analyte search"); routing("serum", "prospective")  # returns "Analyte search"
```

## Evaluation signals

- routing_map.csv exists and contains exactly one row per matrix-scenario combination (4 rows) with at least one available mode each
- Protocol search appears exclusively in plasma rows and never in serum rows
- Analyte search appears under both prospective and retrospective scenarios for both matrices
- Retrospective rows include Sample search, Analyte search and Data filtering mode; prospective rows include Protocol search (plasma only) and Analyte search
- 'Majority Vote' and 'Maximize stable analytes' are absent from routing decisions, appearing only as Protocol-search sub-options

## Limitations

- Protocol search is currently only available for plasma samples, so serum-based prospective sampling recommendations must use Analyte search instead
- In some cases the database underlying ALISTER will not exactly match user conditions and generalization needs to happen, which the hover tooltips in Sample search expose
- App contents are limited to the influences of time delay and temperature during processing of (EDTA) plasma and serum samples

## Evidence

- [methods] Protocol search is currently only available for plasma samples.: "Protocol search is currently only available for plasma samples."
- [readme] ALISTER can provide data for plasma or serum samples. In the first step you need to select which matrix you are working with.: [evidence span withheld — anchor status: external_doc]
- [readme] Retrospective assessment of samples in respect to their pre-analytical stability can be made using:: "Retrospective assessment of samples in respect to their pre-analytical stability can be made using:"
- [readme] In prospective scenarios the app aims to advise on sampling protocols, that ensure stability of analytes of interest.: "In prospective scenarios the app aims to advise on sampling protocols, that ensure stability of analytes of interest."
