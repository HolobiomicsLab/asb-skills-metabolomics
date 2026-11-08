---
name: benchmark-sensitivity-aggregation
description: Use when comparing annotation algorithms across several public LC-MS
  studies and ionization modes and you need a sensitivity mean (SD) summary table,
  scored against known compound references.
when_to_use_negative:
- When only a single assay or one ionization mode is available — aggregation into
  a mean (SD) across datasets is not meaningful.
- When the annotator output is not a ranked top-three list per peak (the aggregation
  is defined over top-three proposals).
tools:
- name: mWISE
  role: source of the four diffusion-based annotation configurations whose top-three
    proposals are scored and aggregated
  repo: https://dev.b2s.club/b2slab/mWISE
- name: xMSannotator
  role: comparison annotator whose top-three proposals are scored on the same peak
    lists and aggregated into the benchmark table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.1c00238
    title: 'mWISE: An Algorithm for Context-Based Annotation of Liquid Chromatography–Mass
      Spectrometry Features through Diffusion in Graphs'
  source_content_hash: sha256:1f3b713671d5f2f04cc31e8d77b459d73ef826af0fa37688737af833f6eddfb4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/b2slab/mWISE
  tools:
  - Diffu Stats
  - FELLA
  - Genomic Ranges
  - R
  - mWISE
  - xMSannotator
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# benchmark-sensitivity-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates top-three annotation-proposal sensitivity across multiple LC-MS assays and ionization modes into a mean (standard deviation) comparison table for metabolite-annotation tools such as mWISE and xMSannotator.

## When to use

Use when comparing annotation algorithms across several public LC-MS studies and ionization modes and you need a sensitivity mean (SD) summary table, scored against known compound references.

## When NOT to use

- When only a single assay or one ionization mode is available — aggregation into a mean (SD) across datasets is not meaningful.
- When the annotator output is not a ranked top-three list per peak (the aggregation is defined over top-three proposals).

## Inputs

- Filtered LC-MS peak lists (features without signal removed; 80% rule applied when intensity equals 0)
- Known compound reference lists per study and ionization mode
- Top-three prioritized candidate lists per peak from each annotator (e.g. mWISE configurations, xMSannotator)

## Outputs

- Sensitivity mean (standard deviation) table across datasets per tool configuration
- Per-study and per-mode sensitivity values (e.g. TP/FN/reference counts)

## How to apply

Run each annotator (e.g. mWISE's four diffusion configurations and xMSannotator) on the same filtered LC-MS peak lists across the studies in both ionization modes, keeping the top three candidates per peak as the final prioritized proposal. Score each tool's top-three proposals against the known compound references per study and mode to compute sensitivity (true positives / number of references). Aggregate sensitivity across studies for each tool configuration and report mean (standard deviation) — in the mWISE benchmark this yielded 0.63 (0.07) for the FELLA/raw-scores configuration versus 0.55 (0.19) for xMSannotator. Assemble the comparison table with all four mWISE configurations alongside xMSannotator so the metric is comparable across methods; also track the same datasets' F1 where reported. Validate that the reproduced mean (SD) values match the article's benchmark table before treating the aggregation as correct.

## Related tools

- **mWISE** (source of the four diffusion-based annotation configurations whose top-three proposals are scored and aggregated) — https://dev.b2s.club/b2slab/mWISE
- **xMSannotator** (comparison annotator whose top-three proposals are scored on the same peak lists and aggregated into the benchmark table)

## Examples

```
Rscript benchmark_sensitivity.R --peaklists assay1,assay2,... --references refs1,refs2,... --tools mWISE,xMSannotator --topk 3
```

## Evaluation signals

- Reproduced FELLA/raw-scores mWISE sensitivity mean (SD) equals 0.63 (0.07) across the benchmark datasets.
- xMSannotator sensitivity mean (SD) equals 0.55 (0.19) on the same datasets.
- All four mWISE configurations and xMSannotator are computed on the identical filtered peak lists and the same top-three proposal format.
- Per-study/mode sensitivity is computed against the known compound references (e.g. Assay 1: 144 TP, 13 FN, 157 references, sensitivity 0.92).
- Aggregated table lists one sensitivity mean (SD) row per tool configuration, comparable across methods.

## Limitations

- mWISE only offers data to annotate in the KEGG database, an important limitation with respect to xMSannotator, which also covers HMDB, Lipid Maps and T3DB — cross-tool comparisons are restricted to KEGG.
- Aggregation uses the top three candidates per peak; configurations that produce fewer candidates are not directly comparable.
- Sensitivity alone does not capture false-positive performance; F1 metrics from the same benchmark should be reported alongside.

## Evidence

- [abstract] Sensitivity aggregation definition: "mWISE shows a sensitivity mean (standard deviation) across data sets of 0.63 (0.07), while xMSannotator achieves a sensitivity of 0.55 (0.19)"
- [results] Top-three proposal rule: "The top three candidates for each peak, if available, are selected as the final prioritized proposal."
- [methods] Input filtering rule: "the LC-MS features without signal were removed and the 80% rule was applied in the cases where the intensity was equal to 0"
- [results] Sensitivity computation example: "Assay 1 144 11605 0 13 0 157 0.92 0.00 0.01 0.01 0.02"
- [results] Database scope limitation: "This is an important limitation with respect to xMSannotator that should be addressed in future versions of mWISE."
