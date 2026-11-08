---
name: prospective-retrospective-scenario-classification
description: 'Use when a metabolomics or lipidomics blood-sample stability question
  must be mapped to an app workflow: decide whether the user is planning a future
  sample collection or assessing already-collected (e.g. biobank) samples, and which
  search mode follows.'
when_to_use_negative:
- The user question concerns analyte-level stability data, thresholds (20%/30% fold
  change), or protocol recommendation logic itself rather than which search mode applies
- The question involves sample matrices beyond (EDTA) plasma and serum, which are
  outside ALISTER's current app contents
- The task is the 'Majority Vote' vs 'Maximize stable analytes' protocol-choice decision
  — that is a downstream Protocol-search option, not a routing decision
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: R
  role: Language in which the routing(matrix, scenario) dispatch function and the
    2x2 grid evaluation over plasma/serum by prospective/retrospective are implemented
  repo: https://github.com/Fraunhofer-ITMP/alister
- name: Shiny
  role: Framework hosting the first-step matrix selection that feeds the scenario-to-search-mode
    dispatch
  repo: https://github.com/Fraunhofer-ITMP/alister
- name: RShiny
  role: The deployed web-application whose menu routing is being encoded and evaluated
  repo: https://github.com/Fraunhofer-ITMP/alister
- name: ALISTER
  role: The target application whose prospective/retrospective scenario structure
    and search modes the dispatcher reproduces
  repo: https://github.com/Fraunhofer-ITMP/alister
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.cca.2024.117858
    title: ALISTER – Application for lipid stability evaluation and research
  source_content_hash: sha256:0f18e594c49eea29c65ac641bcd79873e0ad41b62c194bfc97ac8fda5869b4c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Fraunhofer-ITMP/alister
  tools:
  - ALISTER
  - R
  - RShiny
  - Shiny
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# prospective-retrospective-scenario-classification

## Summary

Classifies a user's pre-analytical question about blood sample stability in metabolomics/lipidomics as either a prospective (sampling-protocol planning) or retrospective (sample-quality assessment) scenario, then routes it to the admissible ALISTER search modes under the plasma-only constraint on Protocol search.

## When to use

Use when a metabolomics or lipidomics blood-sample stability question must be mapped to an app workflow: decide whether the user is planning a future sample collection or assessing already-collected (e.g. biobank) samples, and which search mode follows.

## When NOT to use

- The user question concerns analyte-level stability data, thresholds (20%/30% fold change), or protocol recommendation logic itself rather than which search mode applies
- The question involves sample matrices beyond (EDTA) plasma and serum, which are outside ALISTER's current app contents
- The task is the 'Majority Vote' vs 'Maximize stable analytes' protocol-choice decision — that is a downstream Protocol-search option, not a routing decision

## Inputs

- Matrix selection (plasma or serum) from the app's first step
- Scenario label (prospective planning of sampling vs retrospective assessment of samples)
- Optional user pre-analytical conditions (processing temperature, time delay before/after centrifugation) to check for database generalization fallbacks

## Outputs

- routing_map.csv with columns (matrix, scenario, available_modes), one row per matrix-scenario combination
- Admissible search-mode set per input: Sample search, Analyte search, Data filtering mode, Protocol search
- Flags for mode-generalization fallbacks where exact condition matching fails

## How to apply

Encode the routing specification as a dispatch function routing(matrix, scenario) that returns admissible modes: retrospective scenarios dispatch to Sample search, Analyte search and the Data filtering mode, while prospective scenarios dispatch to Protocol search and Analyte search. Enforce the matrix constraint that Protocol search is currently only available for plasma samples, so a serum selection removes Protocol search from the admissible set and routes users toward the remaining modes. Treat 'Majority Vote' (default) versus 'Maximize stable analytes' as downstream options inside Protocol search, not as routing decisions. Evaluate the dispatcher over the full 2x2 plasma/serum by prospective/retrospective grid, recording mode-generalization fallbacks for cases where user conditions cannot be exactly matched by the underlying database, and write the decision table (matrix, scenario, available_modes) to routing_map.csv.

## Related tools

- **R** (Language in which the routing(matrix, scenario) dispatch function and the 2x2 grid evaluation over plasma/serum by prospective/retrospective are implemented) — https://github.com/Fraunhofer-ITMP/alister
- **Shiny** (Framework hosting the first-step matrix selection that feeds the scenario-to-search-mode dispatch) — https://github.com/Fraunhofer-ITMP/alister
- **RShiny** (The deployed web-application whose menu routing is being encoded and evaluated) — https://github.com/Fraunhofer-ITMP/alister
- **ALISTER** (The target application whose prospective/retrospective scenario structure and search modes the dispatcher reproduces) — https://github.com/Fraunhofer-ITMP/alister

## Examples

```
routing_map <- expand.grid(matrix = c('plasma','serum'), scenario = c('prospective','retrospective')); routing_map$available_modes <- mapply(routing, routing_map$matrix, routing_map$scenario)
```

## Evaluation signals

- routing_map.csv exists and contains one row per matrix-scenario combination (4 rows) with at least one available mode
- Protocol search appears exclusively in plasma rows and never in serum rows
- Analyte search appears under both prospective and retrospective scenarios
- Retrospective rows list Sample search, Analyte search and Data filtering mode; prospective rows list Protocol search (plasma only) and Analyte search
- Rows flagged with generalization fallbacks correspond to conditions the underlying database cannot exactly match, as documented in Sample search hover tooltips

## Limitations

- Protocol search is currently only available for plasma samples; serum prospective planning cannot use it
- In some cases the underlying ALISTER database will not exactly match user conditions, so generalization must occur and inputs are approximated
- Current app contents are limited to the influences of time delay and temperature during processing of (EDTA) plasma and serum samples

## Evidence

- [readme] Retrospective vs prospective routing: "Retrospective assessment of samples in respect to their pre-analytical stability can be made using: - Sample search - Analyte search - Data filtering mode"
- [readme] Prospective routing: "Prospective planning of sampling can be performed using: - Protocol search - Analyte search"
- [readme] Plasma-only constraint: "Protocol search is currently only available for plasma samples."
- [readme] Matrix selection as first step: [evidence span withheld — anchor status: external_doc]
- [intro] Scenario definitions: "In prospective scenarios the app aims to advise on sampling protocols, that ensure stability of analytes of interest."
- [readme] Generalization fallback: "In some cases, the database underlying ALISTER will not exactly match your conditions. In these cases, generalization needs to happen."
