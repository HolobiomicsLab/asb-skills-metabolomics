---
name: timestamp-delay-calculation
description: Use when a sample cohort has date+time stamps for blood collection, centrifugation,
  and freezing and you need the pre-/post-centrifugation delays that drive PRIMA Panel
  performance reports or SPREC assignment.
when_to_use_negative:
- Delays are already measured or provided directly (e.g. a SPREC-coded metadata table)
  — the tool would simply recompute them from redundant timestamps.
- The sample type is not peripheral blood plasma/serum with the collection→centrifugation→freezing
  processing chain the PRIMA models were trained on.
- You only have qualitative delay buckets (e.g. '<2 h') rather than full date+time
  stamps; the panel needs exact differences to reproduce the linear mixed model inputs.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_3520
tools:
- name: PRIMA Panel
  role: Open-source R Shiny app hosting the Tools tab that computes pre-/post-centrifugation
    delays from date-time stamps and feeds them to performance reports
  repo: github.com/funkam/PRIMA
- name: Shiny
  role: Web framework that implements the interactive slider and threshold UI for
    the delay-driven performance report
  repo: github.com/funkam/PRIMA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:a3b03996f83ac57a913958ee5eddd69dcbf73ea42579e2acc1f320142d3f4dfe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PRIMA Panel
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# timestamp-delay-calculation

## Summary

Computes exact pre- and post-centrifugation delays (in minutes) by taking the differences between collection, centrifugation, and freezing date-time stamps, then assigns SPREC classification so sample cohorts can be scored against PRIMA Panel stability models.

## When to use

Use when a sample cohort has date+time stamps for blood collection, centrifugation, and freezing and you need the pre-/post-centrifugation delays that drive PRIMA Panel performance reports or SPREC assignment.

## When NOT to use

- Delays are already measured or provided directly (e.g. a SPREC-coded metadata table) — the tool would simply recompute them from redundant timestamps.
- The sample type is not peripheral blood plasma/serum with the collection→centrifugation→freezing processing chain the PRIMA models were trained on.
- You only have qualitative delay buckets (e.g. '<2 h') rather than full date+time stamps; the panel needs exact differences to reproduce the linear mixed model inputs.

## Inputs

- Cohort table with pre-analytical information: date-time stamps for collection, centrifugation, and freezing (per sample, as accepted by the PRIMA Panel Tools tab)

## Outputs

- Per-sample pre-centrifugation delay in minutes
- Per-sample post-centrifugation delay in minutes
- SPREC classification (letters 2 and 6) per sample
- Input values for the PRIMA Panel 'Single' performance-report slider and mean-percentage-error threshold filtering

## How to apply

In the PRIMA Panel Tools tab, upload the cohort's timestamp table (columns holding collection, centrifugation, and freezing date-time stamps) and compute the pre-centrifugation delay as the difference between collection and centrifugation times, and the post-centrifugation delay as the difference between centrifugation and freezing times; delays are expressed in minutes. Use the resulting delays to assign each sample an SPREC class (letters 2 and 6 in the Data tab ordering) and to set the pre- and post-centrifugation slider values in the Performance Report 'Single' tab, where % thresholds for minor vs. major metabolite changes are user-adjustable. The rationale is that these two computed delays are the same quantities the deposited linear mixed models were fitted against, so predicted concentration changes for NMR-quantified metabolic parameters are only meaningful when derived from them. Cross-check that no timestamp pair yields a negative delay (i.e., centrifugation not before collection) before feeding the table to the models.

## Related tools

- **PRIMA Panel** (Open-source R Shiny app hosting the Tools tab that computes pre-/post-centrifugation delays from date-time stamps and feeds them to performance reports) — github.com/funkam/PRIMA
- **Shiny** (Web framework that implements the interactive slider and threshold UI for the delay-driven performance report) — github.com/funkam/PRIMA

## Examples

```
# In R (PRIMA Panel Tools tab equivalent):
delays <- data.frame(pre = as.numeric(difftime(centrifugation_ts, collection_ts, units = "mins")), post = as.numeric(difftime(freezing_ts, centrifugation_ts, units = "mins")))
```

## Evaluation signals

- No negative delay is produced: centrifugation timestamp ≥ collection timestamp, and freezing timestamp ≥ centrifugation timestamp for every sample.
- Computed delays are in minutes and reproduce the SPREC classification (letters 2 and 6) shown in the PRIMA Data tab when re-derived from the deposited Dresden Integrated Liquid Biobank cohort.
- Slider values in the Performance Report 'Single' tab match the per-sample delays computed in the Tools tab for the same cohort.
- The minor/major change flags in the generated .HTML performance report change consistently when the % thresholds are adjusted, confirming the delays (not the thresholds) drive the metabolite columns.
- Predicted changes fall within the 20% 'stability time point' convention documented for the PRIMA Panel when the delays are at their nominal values.

## Limitations

- No changelog or version information is published for the PRIMA Panel, so delay-calculation behavior may change without notice.
- The models and delay logic were developed and validated on plasma/serum from peripheral blood with a single freeze step; other matrices or processing chains are out of scope.
- Delays derived from coarse or missing timestamps propagate directly into model predictions — the panel does not flag implausible intervals beyond the computed difference.

## Evidence

- [readme] Tools-tab delay derivation from date+time stamps: "Currently it consists of a tool for calculating the pre- and post-centrifugation times from the differences of date+time stamps."
- [readme] Performance report output driven by delays and thresholds: "A table is then generated that higlights a minor and a major change for each metabolite in that given timeframe. The colors, as well as the % threshholds, can be adjusted."
- [intro] Mixed-model prediction driven by computed delays: "These data were then used to generate linear mixed models that allow for prediction and quantification of the changes each metabolic parameter undergoes with prolonged processing delays"
- [abstract] Stability time point = 20% change convention: "The study introduced the concept of a “stability time point”, marking when a metabolite’s concentration changes by 20%"
