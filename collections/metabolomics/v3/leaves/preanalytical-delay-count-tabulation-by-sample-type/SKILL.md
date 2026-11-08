---
name: preanalytical-delay-count-tabulation-by-sample-type
description: Use when you have PRIMA Panel stability time point data (minutes, annotated
  by sample type and pre-/postcentrifugation delay) and need per-sample-type counts
  of parameters stable within a fixed delay threshold.
when_to_use_negative:
- If you need metabolite-level concentration changes over time rather than a summary
  count of stable parameters — use the PRIMA Panel's interactive Data tab or Performance
  Reports instead.
- If your data are raw NMR spectra or a performance report for a single pre-/postcentrifugation
  delay combination rather than a stability time point table annotated by sample type.
- If you are computing pre-/postcentrifugation durations from date+time stamps — use
  the PRIMA Panel Tools tab calculator instead.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3520
tools:
- name: PRIMA Panel
  role: Open-source R Shiny tool whose Data tab provides the SPREC-sorted stability
    time point values (in minutes) split by pre- and postcentrifugation that this
    counting/tabulation is performed on
  repo: https://github.com/funkam/PRIMA
- name: R
  role: Language used to load, threshold, group, and tabulate the stability time point
    table
  repo: https://github.com/funkam/PRIMA
- name: Shiny
  role: Framework hosting the PRIMA Panel Data tab through which stability time points
    are presented and explored
  repo: https://github.com/funkam/PRIMA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:d93c6eb02831f7e6953cbfadca135af6d8d8029526ad369ad215652488768264
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/funkam/PRIMA
  tools:
  - PRIMA Panel
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# preanalytical-delay-count-tabulation-by-sample-type

## Summary

Tabulates, per sample type (EDTA plasma, serum, LiHep plasma), the number of metabolic parameters whose stability time points fall within a chosen pre- or postcentrifugation delay window (e.g. 4 h), using the PRIMA Panel's SPREC-sorted stability time point data.

## When to use

Use when you have PRIMA Panel stability time point data (minutes, annotated by sample type and pre-/postcentrifugation delay) and need per-sample-type counts of parameters stable within a fixed delay threshold.

## When NOT to use

- If you need metabolite-level concentration changes over time rather than a summary count of stable parameters — use the PRIMA Panel's interactive Data tab or Performance Reports instead.
- If your data are raw NMR spectra or a performance report for a single pre-/postcentrifugation delay combination rather than a stability time point table annotated by sample type.
- If you are computing pre-/postcentrifugation durations from date+time stamps — use the PRIMA Panel Tools tab calculator instead.

## Inputs

- Recomputed stability time point table (stability time point in minutes per metabolic parameter, annotated with sample type: EDTA plasma, serum, LiHep plasma, and delay type: pre- vs postcentrifugation, sorted by SPREC classification)
- Delay threshold in minutes (e.g. 240 min = 4 h)

## Outputs

- Summary table of parameter counts with columns delay type, sample type, and count of parameters within the threshold
- Per-cell match/mismatch flags when counts are compared against reference values

## How to apply

Load the recomputed stability time point table in which each metabolic parameter's stability time point (in minutes) is annotated with its sample type (EDTA plasma, serum, LiHep plasma) and delay type (pre- vs postcentrifugation), sorted by SPREC classification as the PRIMA Panel's Data tab does. Convert the stability time points from minutes to hours and apply the delay threshold (e.g. 240 min for the first 4 h), flagging which parameters remain stable within that window for each parameter-sample type-delay type combination. Group the flagged rows by delay type and sample type, then count the distinct metabolic parameters in each of the resulting cells (e.g. six combinations for three sample types × two delay types). Assemble the counts into a summary table with columns for delay type, sample type, and count, and compare each computed count against a reference (here: precentrifugation 26/26/15 and postcentrifugation 26/23/8 for EDTA plasma/serum/LiHep respectively) to verify correctness. The rationale is that stability time points — the delay at which a metabolite's concentration changes by 20% — differ by sample type, so tabulating them per type reveals which parameters survive a given centralized-biobank delay.

## Related tools

- **PRIMA Panel** (Open-source R Shiny tool whose Data tab provides the SPREC-sorted stability time point values (in minutes) split by pre- and postcentrifugation that this counting/tabulation is performed on) — https://github.com/funkam/PRIMA
- **R** (Language used to load, threshold, group, and tabulate the stability time point table) — https://github.com/funkam/PRIMA
- **Shiny** (Framework hosting the PRIMA Panel Data tab through which stability time points are presented and explored) — https://github.com/funkam/PRIMA

## Examples

```
R: subset(stability_tpt, stp_min <= 240) |> dplyr::count(delay_type, sample_type)  # yields pre: EDTA 26, serum 26, LiHep 15; post: EDTA 26, serum 23, LiHep 8
```

## Evaluation signals

- All computed counts must equal the reported reference counts exactly: precentrifugation 26 (EDTA plasma), 26 (serum), 15 (LiHep plasma); postcentrifugation 26 (EDTA plasma), 23 (serum), 8 (LiHep plasma).
- Each count cell must correspond to a unique delay type × sample type combination (six cells for three sample types × two delay types).
- Threshold must be applied in minutes (240 min = 4 h); converting minutes to hours before flagging should yield identical counts.
- Counted parameters must be distinct metabolic parameters within each delay type × sample type group (no duplicate parameter entries inflating counts).
- Summary table must retain the SPREC classification ordering used to sort stability time points in the PRIMA Panel Data tab.

## Limitations

- Counts are only valid for the delay threshold chosen (here 4 h / 240 min); a different threshold will change the parameter counts and cannot be compared against the 26/26/15 and 26/23/8 reference values.
- Results depend on the 20% concentration-change definition of the stability time point; metabolites such as lactic acid are affected by even short delays, so the counts reflect that cutoff rather than absolute stability.
- The skill assumes the stability time point table is pre-annotated with sample type and delay type (SPREC classification); raw PRIMA Panel output without these annotations cannot be tabulated directly.

## Evidence

- [readme] The time-points are sorted according to their SPREC classification: "The Data tab shows different ways of highlighting the different stability time-points in minutes. The time-points are sorted according to their SPREC classification"
- [readme] The data is split according the two different delays: "The data is split according the two different delays (pre- and post-centrifugation"
- [abstract] stability time point - 20% concentration change: "The study introduced the concept of a “stability time point”, marking when a metabolite’s concentration changes by 20%"
- [abstract] PRIMA Panel open-source R Shiny tool: "the authors developed the PRIMA Panel, an open-source R Shiny tool"
- [abstract] Lactic acid affected by short delays: "some metabolites, such as lactic acid, were significantly affected by even short delays, while others remained stable for longer"
