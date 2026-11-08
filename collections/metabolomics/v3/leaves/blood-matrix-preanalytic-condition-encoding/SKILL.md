---
name: blood-matrix-preanalytic-condition-encoding
description: Use when building or extending a pre-analytical stability database from
  published EDTA plasma/serum studies that report fold changes due to time delay and
  temperature during sample processing.
when_to_use_negative:
- The question concerns matrices other than (EDTA) plasma or serum — current app contents
  are limited to these two sample types
- The study effect being encoded is not pre-analytical time delay or temperature during
  processing
- Analyte identifiers cannot be mapped to lipid shorthand or RefMet nomenclature,
  so they cannot be matched against the ALISTER database
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_3361
tools:
- name: R
  role: Used to curate fold-change measurements from published studies, normalize
    analyte identifiers, compute fold changes per condition and export the aggregated
    stability_database.csv
  repo: github.com/Fraunhofer-ITMP/alister
- name: RShiny
  role: Framework for the ALISTER web-application that queries the encoded stability
    database in sample, analyte, protocol and data-filtering search modes
  repo: github.com/Fraunhofer-ITMP/alister
- name: RefMet Database
  role: Source of standardized names for polar metabolites during analyte identifier
    normalization
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.cca.2024.117858
    title: ALISTER – Application for lipid stability evaluation and research
  source_content_hash: sha256:cdb5ebab5013ed0798c4131837cfd1dd8307e088bf48a73bf90a3cf1139fb93e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - RShiny
  - RefMet Database
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# blood-matrix-preanalytic-condition-encoding

## Summary

Curates fold-change measurements from published pre-analytic blood stability studies and encodes each measurement with its matrix (EDTA plasma or serum), processing temperatures and delays before/after centrifugation, so analyte stability can be aggregated, queried and traced to literature.

## When to use

Use when building or extending a pre-analytical stability database from published EDTA plasma/serum studies that report fold changes due to time delay and temperature during sample processing.

## When NOT to use

- The question concerns matrices other than (EDTA) plasma or serum — current app contents are limited to these two sample types
- The study effect being encoded is not pre-analytical time delay or temperature during processing
- Analyte identifiers cannot be mapped to lipid shorthand or RefMet nomenclature, so they cannot be matched against the ALISTER database

## Inputs

- Published pre-analytic stability studies of EDTA plasma and serum reporting fold changes for time-delay and temperature effects
- Lipid shorthand nomenclature for lipid analyte identifiers
- RefMet nomenclature for polar metabolite identifiers

## Outputs

- stability_database.csv containing query conditions, stability classification (stable/unstable at 20% or 30% fold-change thresholds) and literature references per analyte

## How to apply

In R, extract fold-change measurements from published pre-analytic studies of EDTA plasma and serum, keeping time delays before and after centrifugation as separate condition fields. Normalize analyte identifiers using lipid shorthand nomenclature for lipids and RefMet names for polar metabolites, and assign each analyte its main lipid class or RefMet main class. Encode every measurement with the matrix (EDTA plasma or serum), the processing temperature — using distinct pre- and post-centrifugation temperature fields when they varied — and the processing delays. Compute fold changes per condition and flag instability against the default 20% fold-change threshold, expanding the limit to 30% when pre- and post-centrifugation effects are combined. Attach the supporting literature citation to each analyte's stability assessment so database entries remain traceable, then export the aggregated table to stability_database.csv.

## Related tools

- **R** (Used to curate fold-change measurements from published studies, normalize analyte identifiers, compute fold changes per condition and export the aggregated stability_database.csv) — github.com/Fraunhofer-ITMP/alister
- **RShiny** (Framework for the ALISTER web-application that queries the encoded stability database in sample, analyte, protocol and data-filtering search modes) — github.com/Fraunhofer-ITMP/alister
- **RefMet Database** (Source of standardized names for polar metabolites during analyte identifier normalization)

## Examples

```
Rscript curate_stability_data.R --studies published_preanalytic/ --out stability_database.csv
```

## Evaluation signals

- Exported stability_database.csv contains separate condition fields for delays before vs. after centrifugation and distinct pre-/post-centrifugation temperature fields where they varied
- Every analyte row carries a traceable literature citation for its stability assessment
- Instability flags match the thresholds: 20% fold change for single conditions, 30% when pre- and post-centrifugation effects are combined
- Lipid analytes resolve to lipid shorthand nomenclature and polar metabolites to RefMet names, each with an assigned main lipid class or RefMet main class
- Fold changes per condition are reproducible from the source studies' reported measurements

## Limitations

- Current app contents are limited to the influences of time delay and temperature during processing of (EDTA) plasma and serum samples
- When the database does not exactly match user conditions, generalization of input conditions is required and must be surfaced to the user
- Protocol search is currently only available for plasma samples
- Data-driven aggregation requires large amounts of experimental data; the project solicits contributed studies to grow the database

## Evidence

- [other] Encode each measurement with the matrix (EDTA plasma or serum), the processing temperature (with distinct pre- and post-centrifugation temperature fields when they varied) and the processing delays.: [evidence span withheld — anchor status: not_found]
- [other] Compute fold changes per condition and flag instability against the default 20% fold-change threshold, expanding the limit to 30% when pre- and post-centrifugation effects are combined.: [evidence span withheld — anchor status: not_found]
- [methods] By default the critical threshold for assessing instability is set to 20% fold change due to specific pre-analytical conditions.: "By default the critical threshold for assessing instability is set to 20% fold change due to specific pre-analytical conditions."
- [readme] Current app contents are limited to the influences of time delay and temperature during processing of (EDTA) plasma and serum samples: "Current app contents are limited to the influences of time delay and temperature during processing of (EDTA) plasma and serum samples, which have been identified to be among the major pre-analytical"
- [readme] Lipids are named after the lipid shorthand nomenclature, while polar metabolites are named after their RefMet nomenclature.: "Lipids are named after the lipid shorthand nomenclature[^2], while polar metabolites are named after their RefMet nomenclature[^3]."
- [abstract] Data of multiple pre-analytic studies were aggregated into a database.: "Data of multiple pre-analytic studies were aggregated into a database."
