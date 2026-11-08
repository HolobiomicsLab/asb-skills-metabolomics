---
name: csv-dataset-annotation
description: Use when a user has a .csv measurement matrix (samples in rows, analytes
  in columns) of lipids/polar metabolites from (EDTA) plasma or serum, knows the processing
  conditions, and wants analytes flagged as 'unstable' pre-emptively.
when_to_use_negative:
- The goal is prospective planning of sampling — use ALISTER's 'Protocol search' (plasma
  only) or 'Majority Vote'/'Maximize stable analytes' instead of the data filtering
  mode.
- The user wants a detailed stability and sampling-protocol investigation of one specific
  analyte — 'Analyte search' provides literature summaries and direction of fold change
  via 'Details'.
- The pre-analytical question involves influences other than time delay and temperature
  during processing, since ALISTER's contents are limited to (EDTA) plasma and serum
  samples under these two factors.
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3316
- http://edamontology.org/topic_0091
tools:
- name: ALISTER
  role: Web-app providing the data filtering mode; matches uploaded analyte names
    against its stability database and returns the annotated dataset
  repo: https://github.com/Fraunhofer-ITMP/alister
- name: RShiny
  role: Framework in which the ALISTER data filtering web-application is implemented
    and hosted
  repo: https://github.com/Fraunhofer-ITMP/alister
- name: R
  role: Language underlying the RShiny app used to run the filtering query and produce
    the annotated .csv
- name: RefMet Database
  role: Nomenclature reference for naming polar metabolites when matching uploaded
    analyte names against the ALISTER database
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1016/j.cca.2024.117858
    title: ALISTER – Application for lipid stability evaluation and research
  source_content_hash: sha256:c1375bdedbdc9372946d461613d0e2a3a0fe7608c40278915169e3c9a1e99595
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Fraunhofer-ITMP/alister
  tools:
  - ALISTER
  - R
  - RShiny
  - RefMet Database
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3316
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# csv-dataset-annotation

## Summary

ALISTER's data filtering mode: upload a tabular .csv of analyte measurements and annotate each analyte as stable or unstable under user-specified pre-analytical conditions (processing temperature, delays before/after centrifugation), using literature-aggregated blood sample stability data with a default 20% fold-change instability threshold.

## When to use

Use when a user has a .csv measurement matrix (samples in rows, analytes in columns) of lipids/polar metabolites from (EDTA) plasma or serum, knows the processing conditions, and wants analytes flagged as 'unstable' pre-emptively.

## When NOT to use

- The goal is prospective planning of sampling — use ALISTER's 'Protocol search' (plasma only) or 'Majority Vote'/'Maximize stable analytes' instead of the data filtering mode.
- The user wants a detailed stability and sampling-protocol investigation of one specific analyte — 'Analyte search' provides literature summaries and direction of fold change via 'Details'.
- The pre-analytical question involves influences other than time delay and temperature during processing, since ALISTER's contents are limited to (EDTA) plasma and serum samples under these two factors.

## Inputs

- .csv file of measurements in tabular form with sample names in rows and analyte names in columns (an empty table containing only analyte names is valid)
- Processing temperature during sample handling (optionally separate pre- and post-centrifugation temperatures)
- Processing delays before and after centrifugation
- Optional custom stability thresholds via 'Define stability thresholds'

## Outputs

- Annotated .csv dataset (original measurements plus an 'unstable' flag for every analyte exceeding the applied fold-change threshold)
- Saved input conditions
- Pie chart of stability-category percentages among the analytes

## How to apply

Load the user-supplied .csv (tabular form, sample names in rows, analyte names in columns; an empty table containing only analyte names is also valid) and match analyte names against ALISTER's underlying database using lipid shorthand nomenclature for lipids and RefMet nomenclature for polar metabolites. Specify the processing temperature, optionally a different temperature after centrifugation via 'Other temperature after centrifugation', and the processing delays before and after centrifugation; influences are assessed separately and then amalgamated. Classify each analyte against the closest experimental data in the database, generalizing the input conditions, using the default critical instability threshold of 20% fold change, expanded to 30% when conditions are combined (readjustable via 'Define stability thresholds'). Export the annotated version of the dataset in which all analytes evaluated as 'unstable' are flagged, preserving the original measurements and optionally the input conditions.

## Related tools

- **ALISTER** (Web-app providing the data filtering mode; matches uploaded analyte names against its stability database and returns the annotated dataset) — https://github.com/Fraunhofer-ITMP/alister
- **RShiny** (Framework in which the ALISTER data filtering web-application is implemented and hosted) — https://github.com/Fraunhofer-ITMP/alister
- **R** (Language underlying the RShiny app used to run the filtering query and produce the annotated .csv)
- **RefMet Database** (Nomenclature reference for naming polar metabolites when matching uploaded analyte names against the ALISTER database)

## Examples

```
import pandas as pd; df = pd.read_csv('measurements.csv')  # samples in rows, analytes in columns; upload via ALISTER data filtering mode
```

## Evaluation signals

- The annotated output .csv exists, is a valid .csv, and contains all original analyte columns and rows of the uploaded dataset.
- Every analyte classified as exceeding the applied fold-change threshold (20% default, 30% when conditions are combined) carries the 'unstable' flag.
- The 'unstable' flag counts are consistent with the pie-chart stability-category percentages shown for the query.
- Input data is unchanged — the app does not alter or store uploaded measurements; only annotations are added.
- Hovering over each analyte shows how the input conditions were approximated/generalized by available experimental data, with a 'Citation' link to the source literature.

## Limitations

- Current app contents are limited to influences of time delay and temperature during processing of (EDTA) plasma and serum samples.
- When the underlying database does not exactly match the user's conditions, generalization/approximation to the closest experimental data occurs; the interpretation of each match must be checked via the hover tooltip.
- Pre-analytical influences (delays before and after centrifugation) are mostly tested separately in the literature, so the combined assessment is an amalgamation rather than directly measured data.

## Evidence

- [readme] data filtering upload: "First up you need to specify a .csv file for upload into the app environment."
- [readme] annotated output: "Despite still giving you the possibility to save your input conditions, you are presented with an annotated version of your dataset."
- [readme] nomenclature matching: "The analyte names will be automatically matched against the ones in ALISTER´s underlying database."
- [methods] threshold default: "By default the critical threshold for assessing instability is set to 20% fold change due to specific pre-analytical conditions."
- [readme] amalgamated assessment: "In this way, pre-analytical influences will be assessed separately and then amalgamated."
