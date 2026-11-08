---
name: retention-time-library-benchmarking
description: Use when you need to quantify how faithfully an automated LC-MS peak-quality
  filter reproduces manual quality labeling of standard peaks, using precision, recall,
  F1, and misclassification counts.
when_to_use_negative:
- Input is already a curated library of only 'good' adducts (exported_metScribeR_library.csv)
  with no bad/multimodal peaks retained — no negative class exists to compute precision/recall
  against.
- The peaks lack manual 'Good Peak'/'Bad Peak' annotations, leaving no ground-truth
  labels to benchmark the automated filter against.
- You need RT prediction accuracy metrics (e.g. correlation between manual and metScribeR-derived
  RTs) rather than peak-filtering quality metrics — that is a separate validation
  task.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: metScribeR
  role: Generates the automated peak-quality filter calls and exports the with_metrics
    CSV that serves as the prediction set for benchmarking against manual labels
  repo: https://github.com/ncats/metScribeR
- name: R
  role: Computes the confusion counts and precision/recall/F1 metrics from the label
    agreement table
- name: Shiny
  role: Provides the Review Results tab where manual Good Peak / Bad Peak / Multimodal/Indeterminate
    labels are collected as ground truth
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:8121e4c1ca4dab0e7c14772adea8b27af44127dfc70248725d036a574af01ff1
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
  - R
  - Shiny
  - metScribeR
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# retention-time-library-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks metScribeR's automated peak-filtering decisions against manual 'Good Peak'/'Bad Peak' labels on LC-MS authentic-standard data, yielding precision, recall, and F1 metrics that quantify filter quality.

## When to use

Use when you need to quantify how faithfully an automated LC-MS peak-quality filter reproduces manual quality labeling of standard peaks, using precision, recall, F1, and misclassification counts.

## When NOT to use

- Input is already a curated library of only 'good' adducts (exported_metScribeR_library.csv) with no bad/multimodal peaks retained — no negative class exists to compute precision/recall against.
- The peaks lack manual 'Good Peak'/'Bad Peak' annotations, leaving no ground-truth labels to benchmark the automated filter against.
- You need RT prediction accuracy metrics (e.g. correlation between manual and metScribeR-derived RTs) rather than peak-filtering quality metrics — that is a separate validation task.

## Inputs

- exported_metScribeR_library_with_metrics.csv — all 'good', 'bad', and 'multimodal/indeterminate' adducts with peak metrics and manual quality labels
- Manual quality labels (Good Peak / Bad Peak / Multimodal/Indeterminate) collected via the metScribeR Review Results tab
- LC-MS mzML files of authentic standards processed with the benchmark filtering criteria

## Outputs

- filtering_performance_metrics.csv — precision, recall, F1, and FP/FN/TN/TP counts
- Label agreement (confusion) table comparing automated 'good' calls vs manual labels

## How to apply

Load exported_metScribeR_library_with_metrics.csv, which retains all 'good', 'bad', and 'multimodal/indeterminate' adducts together with peak metrics and manual quality labels. Reproduce the automated filter decisions under the benchmark criteria: at least 7 data points per peak in a single EIC, width at half maximum intensity between 0 s and 15 s, maximum 250 peaks per EIC, and minimum peak intensity 10,000 (10 times the background noise level of 1000). Binarize the comparison by treating automated 'good' calls as predictions and manual 'Good Peak'/'Bad Peak' clicks as ground truth, excluding 'Multimodal/Indeterminate' peaks from the positive/negative tally. Compute the confusion counts (TP, FP, FN, TN) from the label agreement table and derive precision, recall, and combined F1. In the article's benchmark against 860 manually labeled peaks (from 743 adducts), this yielded precision 0.93, recall 0.98, F1 0.95, with 43 false positives, 10 false negatives, and 226 true negatives.

## Related tools

- **metScribeR** (Generates the automated peak-quality filter calls and exports the with_metrics CSV that serves as the prediction set for benchmarking against manual labels) — https://github.com/ncats/metScribeR
- **R** (Computes the confusion counts and precision/recall/F1 metrics from the label agreement table)
- **Shiny** (Provides the Review Results tab where manual Good Peak / Bad Peak / Multimodal/Indeterminate labels are collected as ground truth)

## Examples

```
Rscript benchmark_filtering.R --input exported_metScribeR_library_with_metrics.csv --output filtering_performance_metrics.csv
```

## Evaluation signals

- Recomputed precision, recall, and F1 must exactly match the reported benchmark values of 0.93, 0.98, and 0.95
- Confusion counts must equal FP=43, FN=10, TN=226 against the 860-peak manual benchmark (743 adducts)
- Every peak must carry exactly one of 'Good Peak', 'Bad Peak', or 'Multimodal/Indeterminate'; multimodal peaks are excluded from the positive/negative tally
- Automated 'good' call must be reproducible under the stated filter criteria: ≥7 data points per peak in a single EIC, half-max width in (0 s, 15 s), ≤250 peaks per EIC, and intensity ≥10,000
- filtering_performance_metrics.csv must contain precision, recall, F1, and the FP/FN/TN counts as named fields

## Limitations

- 'Multimodal/Indeterminate' peaks are excluded from the positive/negative tally, so the benchmark covers only unimodal manual calls
- Filter thresholds (7 data points, 0–15 s half-max width, 250 peaks per EIC, intensity ≥10,000) were tuned for the NCATS benchmarking LC-MS method and may not transfer to other instruments or chromatographic conditions
- The 10,000 intensity cutoff assumes a background noise level of 1000; different noise levels require recalibration of that threshold

## Evidence

- [results] metScribeR filtering achieved precision of 0.93, recall of 0.98, and a combined F1 score of 0.95: [evidence span withheld — anchor status: not_found]
