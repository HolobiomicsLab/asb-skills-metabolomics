---
name: pearson-correlation-and-rmsd-analysis
description: Use when validating that automated RT assignments match manual RT assignments,
  e.g. comparing metScribeR-derived RTs against manually assigned MassHunter RTs on
  authentic metabolite standards.
when_to_use_negative:
- The input is already a peak-quality label (Good/Bad/Multimodal) — use peak-review
  validation instead of RT correlation.
- The reference RTs are not from a single LC-MS method — RT is not consistently transferable
  across different LC-MS conditions, so cross-method comparison would not be meaningful.
- The goal is MS/MS library building rather than RT benchmarking — metScribeR does
  not require MS/MS data and MoNA MS/MS collection is slow and optional.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: metScribeR
  role: source of the automated RT assignments to be benchmarked; exports RTs via
    export_library_csv for comparison
  repo: https://github.com/ncats/metScribeR
- name: R
  role: environment for computing the Pearson correlation and RMSD and writing rt_accuracy_metrics.csv
- name: MassHunter
  role: manual RT-assignment software providing the reference RTs for the IROA standards
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:cc2d5edf3063cff25fdf44df080cb2568176d8faaf3676c7b48a94fb06ff0f8f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ncats/metScribeR
  repo_url: https://github.com/ncats/metScribeR
  tools:
  - MassHunter
  - R
  - Shiny
  - chromatographR
  - metScribeR
  - mzR
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# pearson-correlation-and-rmsd-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the accuracy of automated retention-time assignments (e.g. metScribeR-derived RTs) against a manual reference set by computing the Pearson correlation coefficient and the root-mean-square deviation (RMSD) of paired RT values.

## When to use

Use when validating that automated RT assignments match manual RT assignments, e.g. comparing metScribeR-derived RTs against manually assigned MassHunter RTs on authentic metabolite standards.

## When NOT to use

- The input is already a peak-quality label (Good/Bad/Multimodal) — use peak-review validation instead of RT correlation.
- The reference RTs are not from a single LC-MS method — RT is not consistently transferable across different LC-MS conditions, so cross-method comparison would not be meaningful.
- The goal is MS/MS library building rather than RT benchmarking — metScribeR does not require MS/MS data and MoNA MS/MS collection is slow and optional.

## Inputs

- metScribeR-derived RT assignments (exported_metScribeR_library.csv from export_library_csv)
- manual RT assignments from MassHunter for the IROA metabolite standards
- join table pairing each standard's automated RT with its manual RT

## Outputs

- Pearson correlation coefficient between manual and automated RTs
- RMSD of RT assignments (seconds)
- rt_accuracy_metrics.csv containing both metrics

## How to apply

Pair each standard's automated RT (exported from metScribeR via export_library_csv as exported_metScribeR_library.csv, after EIC creation, peak detection with the benchmark filters: at least 7 data points per peak, width at half maximum between 0 and 15 s, max 250 peaks per EIC, and intensity above 10,000 vs background noise of 1000) with its manually assigned RT from MassHunter. Compute the Pearson correlation coefficient across all paired RTs and the RMSD of the RT differences, then write both metrics to rt_accuracy_metrics.csv. In the metScribeR benchmark, the accepted values were a correlation of 0.99 and an RMSD of 0.54 s, equivalent to 0.18% of the 5 min column cycle time; a recomputation is correct if it reproduces those values within reasonable numerical tolerance.

## Related tools

- **metScribeR** (source of the automated RT assignments to be benchmarked; exports RTs via export_library_csv for comparison) — https://github.com/ncats/metScribeR
- **R** (environment for computing the Pearson correlation and RMSD and writing rt_accuracy_metrics.csv)
- **MassHunter** (manual RT-assignment software providing the reference RTs for the IROA standards)

## Examples

```
library(metScribeR); setwd('C:/Users/user123/Downloads/metScribeR_extdata_folder'); runMetScribeRShinyApp()
```

## Evaluation signals

- Pearson correlation between manual and metScribeR-derived RTs matches 0.99 within reasonable numerical tolerance
- RT RMSD matches 0.54 s within reasonable numerical tolerance
- RMSD expressed as a fraction of the 5 min column cycle time equals ~0.18%
- rt_accuracy_metrics.csv exists and contains both the correlation and RMSD values
- the set of paired standards matches the IROA standards list after applying the benchmark peak filters (≥7 data points, width at half max 0–15 s, ≤250 peaks per EIC, intensity ≥10,000)

## Limitations

- RT is not consistently transferable or predictable across different laboratory-specific or method-specific LC-MS conditions, so the benchmark only holds within a single LC-MS method.
- Only metrics within 'reasonable numerical tolerance' of the published 0.99/0.54 s values are accepted; exact reproduction depends on the same filtering and peak-picking settings.
- metScribeR does not require MS/MS data and reports an identification probability estimate per adduct, so this skill does not assess MS/MS or identification-probability accuracy.

## Evidence

- [methods] Pearson correlation of manual vs metScribeR-derived RTs matches the reported 0.99: [evidence span withheld — anchor status: not_found]
- [results] RMSD of 0.54 s, equivalent to 0.18% of the 5 min column cycle time: [evidence span withheld — anchor status: not_found]
