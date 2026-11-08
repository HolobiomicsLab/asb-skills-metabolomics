---
name: stability-results-csv-export
description: Use when a completed ALISTER query (Sample search, Analyte search, Data
  filtering mode, or Protocol search) needs a persistent, citable record of stability
  classifications for reporting or downstream filtering.
when_to_use_negative:
- The query has not yet been executed in ALISTER — there is no per-analyte classification
  or reference set to export
- The user needs raw measurement data storage; ALISTER explicitly does not alter or
  store uploaded data, so the export cannot replace the original dataset
- The user wants a sampling protocol recommendation for a prospective study rather
  than a retrospective stability record (use Protocol search export instead of Sample
  search export)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0154
tools:
- name: ALISTER
  role: Web application whose download function produces the .csv export containing
    query, stability output, and references
  repo: https://github.com/Fraunhofer-ITMP/alister
- name: RShiny
  role: Framework in which the export functionality is implemented as part of the
    ALISTER app
  repo: https://github.com/Fraunhofer-ITMP/alister
- name: R
  role: Language used to implement the export and annotation logic
  repo: https://github.com/Fraunhofer-ITMP/alister
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.cca.2024.117858
    title: ALISTER – Application for lipid stability evaluation and research
  source_content_hash: sha256:deb3885a97df054a7e1cd7efe77eec788218115117348c88c69478242691a119
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Fraunhofer-ITMP/alister
  tools:
  - ALISTER
  - R
  - RShiny
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0154
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# stability-results-csv-export

## Summary

Exports a .csv file containing the user's ALISTER query, the per-analyte stability classification, and all relevant references, with a 'Variable explanation' tab that documents column meanings for downstream interpretation.

## When to use

Use when a completed ALISTER query (Sample search, Analyte search, Data filtering mode, or Protocol search) needs a persistent, citable record of stability classifications for reporting or downstream filtering.

## When NOT to use

- The query has not yet been executed in ALISTER — there is no per-analyte classification or reference set to export
- The user needs raw measurement data storage; ALISTER explicitly does not alter or store uploaded data, so the export cannot replace the original dataset
- The user wants a sampling protocol recommendation for a prospective study rather than a retrospective stability record (use Protocol search export instead of Sample search export)

## Inputs

- Completed ALISTER query state (selected compound classes, processing temperature, delays before/after centrifugation, stability thresholds)
- Uploaded .csv dataset with sample names in rows and analyte names in columns (Data filtering mode only)

## Outputs

- .csv file containing the query, per-analyte stability output, and all relevant references
- Annotated version of the uploaded dataset with unstable analytes flagged (Data filtering mode)
- PDF of sampling-protocol flowcharts (Protocol search only)

## How to apply

After a query is executed in ALISTER, use the app's download function to export a .csv file containing the query parameters, the output (per-analyte stable/unstable classification at the applied fold-change threshold, e.g. the default 20% or expanded 30% when conditions are combined), and all relevant references. In Data filtering mode the export differs: it returns an annotated version of the uploaded dataset in which analytes evaluated as 'unstable' from literature data and the user's conditions are flagged in-place. The exported file is accompanied by the 'Variable explanation' tab, which documents the meaning of each column so the output can be interpreted without re-running the query.

## Related tools

- **ALISTER** (Web application whose download function produces the .csv export containing query, stability output, and references) — https://github.com/Fraunhofer-ITMP/alister
- **RShiny** (Framework in which the export functionality is implemented as part of the ALISTER app) — https://github.com/Fraunhofer-ITMP/alister
- **R** (Language used to implement the export and annotation logic) — https://github.com/Fraunhofer-ITMP/alister

## Examples

```
Rscript -e "shiny::runApp('alister'); # within ALISTER: execute Sample search → click 'Download' to export query, output & references as .csv"
```

## Evaluation signals

- The exported .csv contains a query section matching the user's entered conditions (temperature, delays, thresholds)
- Per-analyte rows reflect the stable/unstable classification at the applied fold-change threshold (default 20%, 30% when combined)
- Reference columns contain citations for the stability data used for each analyte
- In Data filtering mode, analytes evaluated as 'unstable' appear flagged in the annotated output dataset
- Column meanings match those documented in the app's 'Variable explanation' tab

## Limitations

- The 'Variable explanation' tab must be consulted to interpret download contents, as column semantics are not self-evident
- In Data filtering mode the export is an annotated variant of the input dataset, not a standalone query-result table
- The export depends on ALISTER's underlying database match/generalization; if conditions were generalized, the CSV records that interpretation but the stability call may be an approximation

## Evidence

- [readme] Downloaded CSV containing query, output, and references: "You can download a .csv-file containing your query, as well as the output and all relevant references. The tab 'Variable explanation' will answer all your questions regarding interpretation of the"
- [readme] Data filtering mode export flags unstable analytes: "All analytes, that were evaluated as 'unstable' based on experimental stability data from literature and your input conditions are flagged as such in this variant of your dataset."
- [readme] Protocol search PDF export of flowcharts: "you are able to download a PDF file, that depicts all possible sampling protocols in straight forward flowcharts."
- [readme] Threshold defaults of 20% and 30%: "By default the critical threshold for assessing instability is set to 20% fold change due to specific pre-analytical conditions. When conditions are combined"
