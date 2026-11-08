---
name: rshiny-app-logic-reimplementation
description: Use when a task requires encoding an app's step-to-step routing rules
  (e.g. matrix choice plasma vs serum determining which search modes are reachable)
  as an executable dispatch function and decision table rather than manual UI walkthrough.
when_to_use_negative:
- The goal is to actually run ALISTER to assess analyte stability or get sampling
  recommendations — use the deployed app at https://itmp.shinyapps.io/alister/ instead
  of reimplementing its routing
- The question concerns stability thresholds or fold-change criteria (e.g. the default
  20% threshold expanded to 30% when conditions are combined) rather than which app
  modes are reachable
- The target app's flow is not documented in a README/tutorial and cannot be reconstructed
  from paper text — reimplementation would be speculative
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3382
tools:
- name: R
  role: Language in which the routing dispatch function and the 2x2 grid evaluation
    are encoded
  repo: github.com/Fraunhofer-ITMP/alister
- name: RShiny
  role: Framework whose app logic (step dispatch between search modes) is being reimplemented
    and tested outside the browser
  repo: github.com/Fraunhofer-ITMP/alister
- name: Shiny
  role: Deployment context of the original ALISTER app whose navigation flow is mirrored
    in the dispatcher
  repo: github.com/Fraunhofer-ITMP/alister
- name: ALISTER
  role: Subject app; its four menus (Sample search, Analyte search, Protocol search,
    Data filtering mode) define the routing targets
  repo: github.com/Fraunhofer-ITMP/alister
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.cca.2024.117858
    title: ALISTER – Application for lipid stability evaluation and research
  source_content_hash: sha256:afc97b59f2d558cd6449dca9e8a9afc4c17680380eae5655bd686c4ed5f5972b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALISTER
  - R
  - RShiny
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3382
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# rshiny-app-logic-reimplementation

## Summary

Reimplements the navigation/routing logic of an RShiny web-app (here ALISTER, the Application for lipid stability evaluation and research) as a testable R dispatcher, so that app flow decisions become inspectable and machine-checkable.

## When to use

Use when a task requires encoding an app's step-to-step routing rules (e.g. matrix choice plasma vs serum determining which search modes are reachable) as an executable dispatch function and decision table rather than manual UI walkthrough.

## When NOT to use

- The goal is to actually run ALISTER to assess analyte stability or get sampling recommendations — use the deployed app at https://itmp.shinyapps.io/alister/ instead of reimplementing its routing
- The question concerns stability thresholds or fold-change criteria (e.g. the default 20% threshold expanded to 30% when conditions are combined) rather than which app modes are reachable
- The target app's flow is not documented in a README/tutorial and cannot be reconstructed from paper text — reimplementation would be speculative

## Inputs

- App flow specification from the paper methods and repository README (menu names, first-step matrix selection, scenario-to-mode mapping)
- Documented routing constraints (e.g. Protocol search restricted to plasma samples)
- Scenario definitions: prospective (sampling protocol advice) and retrospective (stability assessment)

## Outputs

- routing_map.csv with columns matrix, scenario, available_modes and one row per matrix-scenario combination
- R dispatch function routing(matrix, scenario) returning admissible search modes
- Recorded mode-generalization fallbacks for conditions not exactly matched by the underlying database

## How to apply

Read the app's documented flow (README/tutorial plus paper methods) and identify the first discriminating step and downstream branches: for ALISTER the first step is matrix selection between (EDTA) plasma and serum, retrospective scenarios dispatch to Sample search, Analyte search and Data filtering mode, and prospective scenarios dispatch to Protocol search and Analyte search. Implement an R function routing(matrix, scenario) returning the admissible modes, enforcing the plasma-only constraint on Protocol search and treating 'Majority Vote' vs 'Maximize stable analytes' as downstream Protocol-search options rather than routing decisions. Evaluate the dispatcher over the full 2x2 plasma/serum by prospective/retrospective grid, recording mode-generalization fallbacks where user conditions cannot be exactly matched by the underlying database. Write the decision table (columns: matrix, scenario, available_modes) to routing_map.csv and validate it against the documented constraints.

## Related tools

- **R** (Language in which the routing dispatch function and the 2x2 grid evaluation are encoded) — github.com/Fraunhofer-ITMP/alister
- **RShiny** (Framework whose app logic (step dispatch between search modes) is being reimplemented and tested outside the browser) — github.com/Fraunhofer-ITMP/alister
- **Shiny** (Deployment context of the original ALISTER app whose navigation flow is mirrored in the dispatcher) — github.com/Fraunhofer-ITMP/alister
- **ALISTER** (Subject app; its four menus (Sample search, Analyte search, Protocol search, Data filtering mode) define the routing targets) — github.com/Fraunhofer-ITMP/alister

## Examples

```
routing <- function(matrix, scenario) if (scenario=="prospective") setdiff(c("Protocol search","Analyte search"), if(matrix=="serum") "Protocol search" else NULL); routing("serum","prospective")
```

## Evaluation signals

- routing_map.csv exists and contains one row per matrix-scenario combination (2x2 grid) with at least one available mode per row
- Protocol search appears exclusively in plasma rows, per the documented plasma-only constraint
- Analyte search appears under both prospective and retrospective scenarios, per the documented mode overlap
- Serum rows contain no Protocol search, matching the routing away from prospective protocol planning for serum
- Mode-generalization fallbacks are recorded for cases where user input conditions cannot be exactly matched by the underlying database

## Limitations

- Protocol search is currently only available for plasma samples, so prospective routing for serum is incomplete by design
- ALISTER's underlying database may not exactly match user-specified conditions, requiring generalization of input conditions in downstream modes
- App contents are limited to the influences of time delay and temperature during processing of (EDTA) plasma and serum samples, so the routing model does not cover other pre-analytical variables

## Evidence

- [methods] Protocol search is currently only available for plasma samples.: "Protocol search is currently only available for plasma samples."
- [readme] ALISTER can provide data for plasma or serum samples. In the first step you need to select which matrix you are working with.: [evidence span withheld — anchor status: external_doc]
- [readme] In some cases, the database underlying ALISTER will not exactly match your conditions. In these cases, generalization needs to happen.: "In some cases, the database underlying ALISTER will not exactly match your conditions. In these cases, generalization needs to happen."
- [intro] In prospective scenarios the app aims to advise on sampling protocols, that ensure stability of analytes of interest.: "In prospective scenarios the app aims to advise on sampling protocols, that ensure stability of analytes of interest."
- [abstract] Flexible approaches for evaluating these data were implemented in an RShiny-based web-application: "Flexible approaches for evaluating these data were implemented in an RShiny-based web-application"
