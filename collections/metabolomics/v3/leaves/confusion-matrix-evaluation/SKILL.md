---
name: confusion-matrix-evaluation
description: Use when comparing automated peak-filtering calls to manual peak-quality
  labels (e.g. Good/Bad Peak clicks) on LC-MS standard library peaks to quantify filtering
  precision, recall, and F1.
when_to_use_negative:
- Input is already a feature table with no manual 'Good Peak'/'Bad Peak' labels —
  there is no ground truth to binarize against.
- Peaks labeled 'Multimodal/Indeterminate' need to be scored — they are excluded from
  the positive/negative tally in this binarization scheme.
- Evaluating adduct identification (m/z/RT assignment or identification probability)
  rather than peak-quality filtering — this skill only scores the filtering step.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: metScribeR
  role: Produces the automated peak-filtering decisions and manual Good/Bad Peak labels
    that are compared in the confusion matrix; exports the metrics CSV evaluated here.
  repo: https://github.com/ncats/metScribeR
- name: R
  role: Language in which the filtering decisions, binarization, and metric computation
    are reproduced.
- name: Shiny
  role: Interface through which metScribeR users perform the manual Good Peak / Bad
    Peak / Multimodal/Indeterminate review that supplies ground truth.
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:fb786b2250f42531e8408b9afffee523f2c49fd0d1e4ec9471f7df49fa947534
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# confusion-matrix-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Binarize automated peak-filtering decisions against manual 'Good Peak'/'Bad Peak' ground-truth labels for LC-MS standard adducts and derive precision, recall, F1, and FP/FN/TN confusion counts, as benchmarked in the metScribeR IROA standards evaluation.

## When to use

Use when comparing automated peak-filtering calls to manual peak-quality labels (e.g. Good/Bad Peak clicks) on LC-MS standard library peaks to quantify filtering precision, recall, and F1.

## When NOT to use

- Input is already a feature table with no manual 'Good Peak'/'Bad Peak' labels — there is no ground truth to binarize against.
- Peaks labeled 'Multimodal/Indeterminate' need to be scored — they are excluded from the positive/negative tally in this binarization scheme.
- Evaluating adduct identification (m/z/RT assignment or identification probability) rather than peak-quality filtering — this skill only scores the filtering step.

## Inputs

- exported_metScribeR_library_with_metrics.csv containing all 'good', 'bad', and 'multimodal/indeterminate' adducts with peak metrics and manual quality labels
- Manual ground-truth peak quality labels ('Good Peak' / 'Bad Peak' clicks from metScribeR's Review Results tab)
- Reproducible filtering criteria (data points per peak, width at half maximum, peaks per EIC, minimum intensity)

## Outputs

- filtering_performance_metrics.csv with precision, recall, F1, and FP/FN/TN counts
- Confusion matrix counts (TP, FP, FN, TN) over binary-filtered adducts

## How to apply

Load an all-adducts metrics table such as exported_metScribeR_library_with_metrics.csv, which retains 'good', 'bad', and 'multimodal/indeterminate' adducts together with peak metrics and manual quality labels. Reproduce the automated filter decisions under the benchmark criteria: minimum 7 data points per peak in a single EIC, width at half maximum intensity between 0 s and 15 s, a maximum of 250 peaks per EIC, and a minimum peak intensity of 10,000 (10 times the 1000 background noise level). Binarize by treating automated 'good' calls as predictions and manual 'Good Peak'/'Bad Peak' clicks as ground truth, excluding 'Multimodal/Indeterminate' peaks from the positive/negative tally. Compute TP, FP=43, FN=10, TN=226 counts from the label-agreement table, then derive precision (0.93), recall (0.98), and combined F1 (0.95) and write all metrics and counts to filtering_performance_metrics.csv.

## Related tools

- **metScribeR** (Produces the automated peak-filtering decisions and manual Good/Bad Peak labels that are compared in the confusion matrix; exports the metrics CSV evaluated here.) — https://github.com/ncats/metScribeR
- **R** (Language in which the filtering decisions, binarization, and metric computation are reproduced.)
- **Shiny** (Interface through which metScribeR users perform the manual Good Peak / Bad Peak / Multimodal/Indeterminate review that supplies ground truth.)

## Examples

```
Rscript compute_filtering_metrics.R --input exported_metScribeR_library_with_metrics.csv --out filtering_performance_metrics.csv
```

## Evaluation signals

- Recomputed precision must exactly equal 0.93
- Recomputed recall must exactly equal 0.98
- Recomputed combined F1 must exactly equal 0.95
- Confusion counts must be FP=43 (passed filtering but manually low-quality), FN=10 (manually 'good' but failed filtering), TN=226 (removed by both manual labeling and filtering)
- Total evaluated adducts (from 743 adducts with manual quality labels over 860 peaks) must exclude 'Multimodal/Indeterminate' peaks from the tally

## Limitations

- 'Multimodal/Indeterminate' manual labels cannot be scored under this binary evaluation and are excluded.
- Manual labeling is user-driven single-click review, so ground truth depends on the reviewer's confidence rather than an independent assay.
- Benchmark criteria (7 data points, 0–15 s width at half maximum, 250 peaks per EIC, intensity ≥ 10,000) were chosen for the specific IROA standards benchmarking data and may need retuning for other LC-MS methods.

## Evidence

- [results] metScribeR filtering achieved precision of 0.93, recall of 0.98, and a combined F1 score of 0.95: [evidence span withheld — anchor status: not_found]
