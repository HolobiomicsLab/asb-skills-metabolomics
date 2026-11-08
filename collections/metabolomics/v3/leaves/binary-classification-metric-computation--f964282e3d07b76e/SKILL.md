---
name: binary-classification-metric-computation
description: Use when an automated per-item quality decision (e.g. metScribeR peak
  filtering on 860 adduct peaks) must be benchmarked against manual binary ground-truth
  labels with precision/recall/F1.
when_to_use_negative:
- When the input library export only contains 'good' adducts (exported_metScribeR_library.csv)
  — the metrics file with 'bad' and 'multimodal/indeterminate' entries is required
  to form the confusion matrix.
- When the evaluation task is multi-class (e.g. ranking three label classes simultaneously)
  rather than a binary quality decision — this skill computes only binary precision/recall/F1.
- When manual labels are not available for a peak subset, e.g. if 'Multimodal/Indeterminate'
  peaks must also be scored — the benchmark protocol excludes them from the positive/negative
  tally.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_2269
tools:
- name: metScribeR
  role: Provides the automated peak filtering whose 'good'/'bad' calls are binarized
    as predictions, and exports the metrics CSV used as input
  repo: https://github.com/ncats/metScribeR
- name: R
  role: Computation environment used to reproduce filter decisions and calculate precision/recall/F1
    from the confusion counts
- name: Shiny
  role: UI through which metScribeR is run to obtain the filtered peak set under review
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.jproteome.5c00548
    title: 'MetScribeR: A Semiautomated Tool for Data Processing of In-House LC-MS
      Metabolite Reference Libraries'
  source_content_hash: sha256:c40f8d404995870b6a197830cf79a8039fbaec97606fd9a3f47950fece6156cb
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# binary-classification-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes precision, recall, and F1 from the confusion counts of an automated binary classifier — here metScribeR's peak filtering — against manual 'Good Peak'/'Bad Peak' quality labels for metabolite-standard adduct peaks, and records them with FP/FN/TN counts for reproducible benchmarking.

## When to use

Use when an automated per-item quality decision (e.g. metScribeR peak filtering on 860 adduct peaks) must be benchmarked against manual binary ground-truth labels with precision/recall/F1.

## When NOT to use

- When the input library export only contains 'good' adducts (exported_metScribeR_library.csv) — the metrics file with 'bad' and 'multimodal/indeterminate' entries is required to form the confusion matrix.
- When the evaluation task is multi-class (e.g. ranking three label classes simultaneously) rather than a binary quality decision — this skill computes only binary precision/recall/F1.
- When manual labels are not available for a peak subset, e.g. if 'Multimodal/Indeterminate' peaks must also be scored — the benchmark protocol excludes them from the positive/negative tally.

## Inputs

- exported_metScribeR_library_with_metrics.csv — contains all 'good', 'bad', and 'multimodal/indeterminate' adducts with peak metrics and manual quality labels
- manual quality labels for 860 peaks (from 743 adducts), recorded as 'Good Peak', 'Bad Peak', or 'Multimodal/Indeterminate'
- automated filter parameters: min 7 data points/peak in a single EIC, width at half max 0–15 s, ≤250 peaks/EIC, min intensity 10,000 (10× noise level 1000)

## Outputs

- filtering_performance_metrics.csv with precision, recall, F1, and FP/FN/TN counts
- label-agreement/confusion table (TP=467, FP=43, FN=10, TN=226 for the metScribeR benchmark)

## How to apply

Load the metScribeR export that retains all 'good', 'bad', and 'multimodal/indeterminate' adducts (exported_metScribeR_library_with_metrics.csv) together with manual quality labels for the LC-MS standards data. Reproduce the filter decisions under the benchmark criteria — minimum 7 data points per peak in a single EIC, width at half maximum intensity greater than 0 s and less than 15 s, at most 250 peaks per EIC, and minimum peak intensity of 10,000 (10× the background noise level of 1000) — and binarize: automated 'good' calls are predictions, manual 'Good Peak'/'Bad Peak' clicks are ground truth, while 'Multimodal/Indeterminate' peaks are excluded from the positive/negative tally. Compute the confusion counts (TP, FP, FN, TN) from the label-agreement table, then derive precision = TP/(TP+FP), recall = TP/(TP+FN), and combined F1 = 2·P·R/(P+R), writing all metrics and counts to a CSV (filtering_performance_metrics.csv). For the metScribeR benchmark, 860 peaks (from 743 adducts) yielded precision 0.93, recall 0.98, F1 0.95 with FP=43, FN=10, TN=226; exact reproduction of these values validates the computation.

## Related tools

- **metScribeR** (Provides the automated peak filtering whose 'good'/'bad' calls are binarized as predictions, and exports the metrics CSV used as input) — https://github.com/ncats/metScribeR
- **R** (Computation environment used to reproduce filter decisions and calculate precision/recall/F1 from the confusion counts)
- **Shiny** (UI through which metScribeR is run to obtain the filtered peak set under review)

## Examples

```
Rscript -e 'd<-read.csv("exported_metScribeR_library_with_metrics.csv"); bin<-d[d$manual_label%in%c("Good Peak","Bad Peak"),]; p<-bin$automated_good==TRUE; g<-bin$manual_label=="Good Peak"; TP<-sum(p&g); FP<-sum(p&!g); FN<-sum(!p&g); TN<-sum(!p&!g); pr<-TP/(TP+FP); rec<-TP/(TP+FN); f1<-2*pr*rec/(pr+rec); write.csv(data.frame(TP,FP,FN,TN,precision=pr,recall=rec,f1=f1),"filtering_performance_metrics.csv",row.names=FALSE)'
```

## Evaluation signals

- Recomputed precision, recall, and F1 exactly match the benchmark values 0.93, 0.98, and 0.95.
- Recomputed FP, FN, and TN counts exactly match 43, 10, and 226.
- Total peaks scored (TP+FP+FN+TN) equals 860, and 'Multimodal/Indeterminate' peaks are absent from the confusion counts.
- All four metrics plus counts appear in filtering_performance_metrics.csv without missing or non-numeric entries.
- Filter decisions reproduce under the stated thresholds: ≥7 data points, 0–15 s width at half max, ≤250 peaks/EIC, intensity ≥10,000.

## Limitations

- 'Multimodal/Indeterminate' peaks are excluded from the binary tally, so the metric does not measure performance on ambiguous peaks.
- Ground truth is manual single-click 'Good Peak'/'Bad Peak' review by the user, so precision/recall are bounded by human labeling accuracy.
- Thresholds (noise level 1000, min intensity 10,000, ≤250 peaks/EIC) were chosen for the benchmarking LC-MS data and may not generalize to other instruments or methods.
- MS/MS data is not used or required; the metric evaluates MS1/RT-based peak quality only.

## Evidence

- [results] Against manual quality labels for 860 peaks (from 743 adducts), metScribeR filtering achieved precision of 0.93, recall of 0.98, and a combined F1 score of 0.95.: [evidence span withheld — anchor status: not_found]
