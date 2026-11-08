---
name: retention-time-error-metric-computation
description: Use when evaluating LC–MS RT prediction models (e.g. METLIN SMRT 25%
  hold-out molecules with experimental vs DLM-predicted RT in seconds) and you must
  report per-molecule relative errors and decide whether to exclude non-retained compounds
  (RT < ~60 s).
when_to_use_negative:
- You are ranking candidate identities by predicted RT from accurate-mass search (top-3
  ranking task) rather than quantifying prediction error
- Your RT data are not experimentally acquired reverse-phase times in seconds (e.g.
  you only have normalized or index-based RT with no interpretable elution-time cutoff
  for non-retained molecules)
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2258
tools:
- name: pandas
  role: Load and filter the SMRT validation set into retained vs non-retained subsets
    before aggregation
- name: NumPy
  role: Compute per-molecule relative errors |predicted − experimental| / experimental
    and their mean/median aggregates
- name: Python
  role: Scripting environment for the filtering and metric computation workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-13680-7
    title: The METLIN small molecule dataset for machine learning-based retention
      time prediction
  source_content_hash: sha256:8fdf41bc4c687ba5b269c1fea40a3cd51fc42d8ff4471a332f68269265049edd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NumPy
  - Python
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# retention-time-error-metric-computation

## Summary

Computes relative retention time (RT) prediction errors |predicted − experimental| / experimental from a deep-learning RT model, on both the full validation set and a retained-molecules-only subset, to expose the mean–median gap (e.g. 47.8% vs 4.6% median) introduced by non-retained molecules eluting before the gradient start.

## When to use

Use when evaluating LC–MS RT prediction models (e.g. METLIN SMRT 25% hold-out molecules with experimental vs DLM-predicted RT in seconds) and you must report per-molecule relative errors and decide whether to exclude non-retained compounds (RT < ~60 s).

## When NOT to use

- You are ranking candidate identities by predicted RT from accurate-mass search (top-3 ranking task) rather than quantifying prediction error
- Your RT data are not experimentally acquired reverse-phase times in seconds (e.g. you only have normalized or index-based RT with no interpretable elution-time cutoff for non-retained molecules)

## Inputs

- Validation-set table of the 25% hold-out molecules from the METLIN SMRT dataset with experimental RT and DLM-predicted RT in seconds
- Chromatographic method (CM) gradient start time / peak width (20–30 s) defining the non-retained threshold (~60 s)

## Outputs

- Mean and median relative RT error on the full validation set (expected ~47.8% mean / ~4.6% median)
- Mean and median relative RT error on the retained-only subset (expected ~6.8% mean / ~4.5% median)
- Per-molecule relative error vector |predicted − experimental| / experimental

## How to apply

Load the validation table of experimental and deep-learning-model (DLM) predicted RTs in seconds for the 25% hold-out molecules from the METLIN SMRT dataset. Classify each molecule as retained or non-retained, flagging compounds that elute before the start of the gradient (typically within the first minute, RT < ~60 s) as non-retained, based on the chromatographic method's peak width (20–30 s in this reverse-phase setup). Compute per-molecule relative errors |predicted − experimental| / experimental with NumPy/pandas, aggregating both mean and median on the full set and on the retained-only subset. The rationale is that non-retained molecules inflate the mean far more than the median because their experimental RT is near zero, so the retained-only filter closes the mean–median gap; the decision point is to report the retained-only metrics (6.8% mean / 4.5% median) alongside the all-molecule metrics (47.8% mean / 4.6% median) rather than replacing them silently.

## Related tools

- **pandas** (Load and filter the SMRT validation set into retained vs non-retained subsets before aggregation)
- **NumPy** (Compute per-molecule relative errors |predicted − experimental| / experimental and their mean/median aggregates)
- **Python** (Scripting environment for the filtering and metric computation workflow)

## Examples

```
rel = (abs(pred - exp) / exp); retained = exp >= 60; print(rel.mean(), rel.median(), rel[retained].mean(), rel[retained].median())
```

## Evaluation signals

- Retained-only mean and median relative errors reproduce the paper values 6.8% and 4.5% within a small tolerance
- Full-set metrics reproduce the reported gap: mean ≫ median (47.8% vs 4.6%), confirming non-retained molecules drive the mean
- Retained-only mean error is markedly lower than the all-molecule mean error
- The non-retained cutoff (~60 s, first minute of gradient) is consistent with the 20–30 s chromatographic peak width benchmark
- Both subset and full-set metrics are reported, not just the filtered numbers

## Limitations

- The relative-error metric is unstable for non-retained molecules whose experimental RT is near zero, so the all-molecule mean (47.8%) is dominated by a small subset rather than reflecting model accuracy
- Metrics are tied to the specific reverse-phase chromatographic method; the paper notes the ability to project predicted RT onto other CMs must be analyzed separately, so thresholds like the ~60 s cutoff do not transfer directly
- The ~60 s non-retained threshold is approximate and depends on gradient start and peak width of the method used

## Evidence

- [abstract] SMRT dataset scope: "an experimentally acquired reverse-phase chromatography retention time dataset covering up to 80,038 small molecules"
- [intro] Model evaluation scope: "We analyzed the model’s predictive ability along with the ability to project predicted RT onto other CMs"
