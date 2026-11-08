---
name: lipid-metabolite-nomenclature-matching
description: Use when a user's metabolomics/lipidomics dataset (samples as rows, analytes
  as columns) contains analyte names that must be reconciled with a curated stability
  database before stability assessment or data filtering.
when_to_use_negative:
- The input already uses ALISTER's own database identifiers or has already been matched
  — re-matching adds nothing
- 'The analytes fall outside ALISTER''s scope: only (EDTA) plasma and serum processing
  effects of time delay and temperature are covered'
- The goal is prospective protocol recommendation (Protocol search) rather than reconciling
  an existing dataset's analyte names
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_0153
tools:
- name: ALISTER
  role: Web-app whose underlying database performs the automatic analyte name matching
    during .csv upload
  repo: https://github.com/Fraunhofer-ITMP/alister
- name: RefMet Database
  role: Nomenclature reference for naming polar metabolites so they match the ALISTER
    database
- name: RShiny
  role: Framework in which the RShiny-based ALISTER web-application implements the
    matching within its upload/search modes
  repo: https://github.com/Fraunhofer-ITMP/alister
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.cca.2024.117858
    title: ALISTER – Application for lipid stability evaluation and research
  source_content_hash: sha256:f3464a6805eed441514b1c96120ff9eb4987973e1fdd527ccc3083fef42d26c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Fraunhofer-ITMP/alister
  tools:
  - ALISTER
  - R
  - RShiny
  - RefMet Database
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_0153
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# lipid-metabolite-nomenclature-matching

## Summary

Maps user-supplied analyte names in an uploaded .csv table onto the ALISTER underlying database by matching lipids via lipid shorthand nomenclature and polar metabolites via RefMet nomenclature, so pre-analytical stability data can be joined to the user's own measurements.

## When to use

Use when a user's metabolomics/lipidomics dataset (samples as rows, analytes as columns) contains analyte names that must be reconciled with a curated stability database before stability assessment or data filtering.

## When NOT to use

- The input already uses ALISTER's own database identifiers or has already been matched — re-matching adds nothing
- The analytes fall outside ALISTER's scope: only (EDTA) plasma and serum processing effects of time delay and temperature are covered
- The goal is prospective protocol recommendation (Protocol search) rather than reconciling an existing dataset's analyte names

## Inputs

- .csv file in tabular form with sample names as rows and analyte names as columns (may be empty except for analyte names)
- Lipid names formatted per lipid shorthand nomenclature
- Polar metabolite names formatted per RefMet nomenclature

## Outputs

- Matched analyte list aligned to ALISTER's underlying database
- Per-analyte stability assessment downstream (annotated dataset with 'unstable' flags in filtered_dataset_annotated.csv for the Data filtering mode)

## How to apply

Require the input as a .csv in tabular form with sample names in rows and analyte names in columns (an empty table containing only analyte names is also valid); the ALISTER app displays it transposed but does not alter or store it. Automatically match each analyte name against ALISTER's underlying database, using lipid shorthand nomenclature for lipids and RefMet nomenclature for polar metabolites, so compound classes align with the database's classification (main lipid class for lipids, RefMet main class for polar metabolites). The match step feeds downstream pre-analytical queries — Sample search, Analyte search, or Data filtering mode — where temperature, processing delays before/after centrifugation, and stability thresholds (default 20% fold change, 30% when conditions are combined) are applied. Only matched analytes can receive a stability category, so unmatched or non-standard names yield no stability assessment and must be renamed to the appropriate nomenclature before re-uploading.

## Related tools

- **ALISTER** (Web-app whose underlying database performs the automatic analyte name matching during .csv upload) — https://github.com/Fraunhofer-ITMP/alister
- **RefMet Database** (Nomenclature reference for naming polar metabolites so they match the ALISTER database)
- **RShiny** (Framework in which the RShiny-based ALISTER web-application implements the matching within its upload/search modes) — https://github.com/Fraunhofer-ITMP/alister

## Evaluation signals

- Every analyte column in the uploaded .csv resolves to an entry in ALISTER's underlying database (no unmatched names) when names follow lipid shorthand / RefMet conventions
- The uploaded file is tabular with sample names as rows and analyte names as columns, as required by the app despite its transposed display
- In Data filtering mode, downstream filtered_dataset_annotated.csv retains all original analyte columns, confirming matching preserved the dataset structure
- Each matched analyte carries a stability category consistent with the pie-chart percentages and the applied 20%/30% fold-change threshold

## Limitations

- Current app contents are limited to influences of time delay and temperature during processing of (EDTA) plasma and serum samples
- Where the database does not exactly match the user's conditions, generalization of input conditions happens — hovering over each analyte shows how input was interpreted
- Names not in lipid shorthand or RefMet nomenclature will not be automatically matched against the database

## Evidence

- [readme] The analyte names will be automatically matched against the ones in ALISTER´s underlying database. Lipids are named after the lipid shorthand nomenclature, while polar metabolites are named after their RefMet nomenclature.: "The analyte names will be automatically matched against the ones in ALISTER´s underlying database. Lipids are named after the lipid shorthand nomenclature"
- [readme] Despite being displayed in a transposed way, the app requires that the file contains the input data in tabular form, with sample names in the rows and analyte names in the columns.: "the app requires that the file contains the input data in tabular form, with sample names in the rows and analyte names in the columns"
- [readme] Analytes are categorized by their main lipid class (when lipids) or their main class according to the RefMet database (for polar metabolites).: "Analytes are categorized by their main lipid class (when lipids) or their main class according to the RefMet database"
- [abstract] Data of multiple pre-analytic studies were aggregated into a database.: "Data of multiple pre-analytic studies were aggregated into a database."
