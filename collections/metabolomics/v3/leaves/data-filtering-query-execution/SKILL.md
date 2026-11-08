---
name: data-filtering-query-execution
description: Use when a user has measured lipids/polar metabolites under known pre-analytical
  conditions and wants analytes flagged or removed as 'unstable' based on literature
  stability data.
when_to_use_negative:
- The goal is prospective sampling planning — use ALISTER's 'Protocol search' instead
  (which is also only available for plasma samples)
- The interest is detailed stability information for one specific analyte — use 'Analyte
  search' rather than filtering a whole dataset
- The matrix is not plasma or serum — ALISTER's contents are limited to (EDTA) plasma
  and serum samples
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ALISTER
  role: Web application (RShiny) providing the data filtering mode, stability database,
    and annotated .csv export
  repo: https://github.com/Fraunhofer-ITMP/alister
- name: RShiny
  role: Framework in which the ALISTER data filtering web-app is implemented and hosted
    on shinyapps.io
  repo: https://github.com/Fraunhofer-ITMP/alister
- name: R
  role: Implementation language of the ALISTER web-application
  repo: https://github.com/Fraunhofer-ITMP/alister
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.cca.2024.117858
    title: ALISTER – Application for lipid stability evaluation and research
  source_content_hash: sha256:96b2ddd5e8f9d0f252745302d65528a2b3cba924a2fdef0f62f4f77b08eb2ae4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Fraunhofer-ITMP/alister
  tools:
  - ALISTER
  - R
  - RShiny
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# data-filtering-query-execution

## Summary

Runs ALISTER's retrospective data filtering mode, which uploads a user dataset (.csv, samples as rows, analytes as columns), matches analyte names against ALISTER's stability database, and flags analytes whose measurements are potentially unsuitable under user-specified pre-analytical conditions (processing temperature, delays before/after centrifugation).

## When to use

Use when a user has measured lipids/polar metabolites under known pre-analytical conditions and wants analytes flagged or removed as 'unstable' based on literature stability data.

## When NOT to use

- The goal is prospective sampling planning — use ALISTER's 'Protocol search' instead (which is also only available for plasma samples)
- The interest is detailed stability information for one specific analyte — use 'Analyte search' rather than filtering a whole dataset
- The matrix is not plasma or serum — ALISTER's contents are limited to (EDTA) plasma and serum samples

## Inputs

- .csv file of measurements in tabular form: sample names as rows, analyte names as columns (empty table with only analyte names also valid)
- Matrix type: (EDTA) plasma or serum
- Processing temperature (optional different temperature after centrifugation)
- Processing delays before and after centrifugation
- Optional custom stability thresholds via 'Define stability thresholds'

## Outputs

- Annotated dataset .csv (filtered_dataset_annotated.csv) preserving original measurements plus 'unstable' flags
- Pie chart of stability-category percentages across the uploaded analytes
- Citations of literature underlying each analyte's stability assessment (via 'Citation')

## How to apply

Specify a .csv file for upload into the ALISTER app environment (https://itmp.shinyapps.io/alister/), ensuring it is tabular with sample names in rows and analyte names in columns (an empty table of only analyte names is valid). Match analyte names against ALISTER's underlying database using lipid shorthand nomenclature for lipids and RefMet nomenclature for polar metabolites. Input the processing temperature (optionally different before/after centrifugation via 'Other temperature after centrifugation') and processing delays before and after centrifugation, which are assessed separately and then amalgamated. Classify stability using the default 20% fold-change critical threshold, expanded to 30% when conditions are combined, unless custom 'Define stability thresholds' limits are set. Export the annotated dataset as a .csv in which every analyte evaluated as 'unstable' from experimental literature data is flagged.

## Related tools

- **ALISTER** (Web application (RShiny) providing the data filtering mode, stability database, and annotated .csv export) — https://github.com/Fraunhofer-ITMP/alister
- **RShiny** (Framework in which the ALISTER data filtering web-app is implemented and hosted on shinyapps.io) — https://github.com/Fraunhofer-ITMP/alister
- **R** (Implementation language of the ALISTER web-application) — https://github.com/Fraunhofer-ITMP/alister

## Examples

```
import pandas as pd; df = pd.read_csv('measurements.csv', index_col=0)  # samples=rows, analytes=cols; upload in ALISTER Data filtering mode at https://itmp.shinyapps.io/alister/
```

## Evaluation signals

- filtered_dataset_annotated.csv exists and is a valid .csv containing all original analyte columns
- Every analyte exceeding the applied fold-change threshold (20% default, 30% when conditions are combined) carries the 'unstable' flag
- 'unstable' flags are consistent with the pie-chart stability-category percentages shown in the app
- Uploaded analyte names were matched to the database using lipid shorthand nomenclature (lipids) and RefMet nomenclature (polar metabolites)
- Hovering over each analyte reveals how the user's input conditions were approximated by available experimental data, allowing spot-checking of generalizations

## Limitations

- App contents are limited to the influences of time delay and temperature during processing of (EDTA) plasma and serum samples
- When the database does not exactly match the user's conditions, generalization of conditions occurs (visible by hovering over each analyte)
- Influence of delay before and after centrifugation is tested separately in most experiments but is amalgamated here, which is why the combined threshold is expanded to 30%

## Evidence

- [methods] Data filtering mode  [section=methods; evidence='The data filtering mode is used in the following way: 1. First up you need to specify a .csv file for upload into the app environment.']: "The data filtering mode is used in the following way: 1. First up you need to specify a .csv file for upload into the app environment."
- [readme] the app requires that the file contains the input data in tabular form, with sample names in the rows and analyte names in the columns: "the app requires that the file contains the input data in tabular form, with sample names in the rows and analyte names in the columns"
- [readme] Lipids are named after the lipid shorthand nomenclature, while polar metabolites are named after their RefMet nomenclature: "Lipids are named after the lipid shorthand nomenclature[^2], while polar metabolites are named after their RefMet nomenclature[^3]."
- [readme] pre-analytical influences will be assessed separately and then amalgamated: "In this way, pre-analytical influences will be assessed separately and then amalgamated."
- [methods] By default the critical threshold for assessing instability is set to 20% fold change due to specific pre-analytical conditions. When conditions are combined: "By default the critical threshold for assessing instability is set to 20% fold change due to specific pre-analytical conditions. When conditions are combined"
- [readme] All analytes, that were evaluated as 'unstable' based on experimental stability data from literature and your input conditions are flagged as such: "All analytes, that were evaluated as 'unstable' based on experimental stability data from literature and your input conditions are flagged as such in this variant of your dataset."
