---
name: metabolite-stability-time-point-extraction
description: Use when you have metabolite concentrations measured under controlled
  pre- or postcentrifugation delays across tube types (EDTA, LiHep, serum) and need
  per-metabolite 20%-change stability thresholds to QC a biobank cohort.
when_to_use_negative:
- The input cohort has no recorded pre-/postcentrifugation delay times or timestamps
  — the models cannot predict concentration change without delay information
- Samples were not analyzed by NMR spectroscopy under the tube types covered by the
  models (EDTA plasma, LiHep plasma, serum) — SPREC-sorted stability points do not
  transfer to other matrices or platforms
- You only need a binary pass/fail QC decision for a single sample rather than per-metabolite
  stability thresholds; Bruker's BiobankQC is the narrower alternative the PRIMA Panel
  extends
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3292
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Environment for loading the delay-experiment tables and fitting linear mixed
    models (donor random intercept, cell-count random slope) used to derive 20%-change
    stability time points
  repo: github.com/funkam/PRIMA
- name: PRIMA Panel
  role: Interactive data-exploration and reporting tool whose Data tab displays the
    stability time points (sorted by SPREC classification) and whose Single tab generates
    performance reports for a cohort given pre-/postcentrifugation times
  repo: github.com/funkam/PRIMA
- name: Shiny
  role: Framework hosting the PRIMA Panel, both locally and at the permanently hosted
    Shinyapps.io deployment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:ce7cea4e5299b87e1f6e8cea0fddd2b29d1f1ce5affd1b4790922bfff99b46a4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PRIMA
  - PRIMA Panel
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3292
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# metabolite-stability-time-point-extraction

## Summary

Extracts the 'stability time point' for each metabolic parameter — the modeled time at which its NMR-quantified concentration changes by 20% — under pre- and postcentrifugation delays in EDTA plasma, LiHep plasma, and serum, using linear mixed models with donor random intercepts and cell-count random slopes.

## When to use

Use when you have metabolite concentrations measured under controlled pre- or postcentrifugation delays across tube types (EDTA, LiHep, serum) and need per-metabolite 20%-change stability thresholds to QC a biobank cohort.

## When NOT to use

- The input cohort has no recorded pre-/postcentrifugation delay times or timestamps — the models cannot predict concentration change without delay information
- Samples were not analyzed by NMR spectroscopy under the tube types covered by the models (EDTA plasma, LiHep plasma, serum) — SPREC-sorted stability points do not transfer to other matrices or platforms
- You only need a binary pass/fail QC decision for a single sample rather than per-metabolite stability thresholds; Bruker's BiobankQC is the narrower alternative the PRIMA Panel extends

## Inputs

- Delay-experiment data tables from github.com/funkam/PRIMA (metabolite concentrations per tube type: EDTA plasma, LiHep plasma, serum)
- Per-sample pre- and postcentrifugation delay times
- Donor identifiers and cell counts (random-effect covariates)

## Outputs

- results_table.csv with recomputed stability time points per metabolic parameter, tube type, and delay type (pre-/postcentrifugation)
- Per-parameter 20%-change thresholds in hours/minutes (e.g., lactic acid: 0.41 h pre-EDTA; 0.81/1.63/4.3 h postcentrifugation in EDTA/LiHep/serum)
- Optional .HTML or .csv performance report from the PRIMA Panel 'Single' tab flagging minor/major changes per metabolite in a chosen delay window

## How to apply

Download the original delay-experiment data tables from the PRIMA GitHub repository (github.com/funkam/PRIMA) and structure them per metabolic parameter with tube type, pre-/postcentrifugation delay times, donor identifiers, and cell counts. Fit a linear mixed model per parameter — donor as random intercept and cell count as random slope — to predict concentration change over prolonged processing delays. From each model, extract the modeled time at which the concentration changes by 20% (the stability time point), separately for pre- and postcentrifugation delays, and report values in hours/minutes per parameter and tube type. Validate against reported anchors: lactic acid 0.41 h precentrifugation in EDTA plasma (~25 min) and 0.81 h (EDTA), 1.63 h (LiHep), 4.3 h (serum) postcentrifugation, plus the lactic acid/glucose ratio change of 21–32 min. The PRIMA Panel Data tab presents these stability time points in minutes, sorted by SPREC classification, so recomputed values can also be cross-checked against the app's lollipop plots or exported performance reports.

## Related tools

- **R** (Environment for loading the delay-experiment tables and fitting linear mixed models (donor random intercept, cell-count random slope) used to derive 20%-change stability time points) — github.com/funkam/PRIMA
- **PRIMA Panel** (Interactive data-exploration and reporting tool whose Data tab displays the stability time points (sorted by SPREC classification) and whose Single tab generates performance reports for a cohort given pre-/postcentrifugation times) — github.com/funkam/PRIMA
- **Shiny** (Framework hosting the PRIMA Panel, both locally and at the permanently hosted Shinyapps.io deployment)

## Examples

```
lmer(concentration ~ delay + (1 | donor) + (0 + cell_count | donor), data = delay_table) # per parameter & tube type; extract delay at which fitted concentration changes by 20%
```

## Evaluation signals

- Recomputed lactic acid precentrifugation stability time point reproduces 0.41 h in EDTA plasma (i.e., ~25 min at room temperature) within the table's reported precision
- Recomputed lactic acid postcentrifugation stability time points reproduce 0.81 h (EDTA), 1.63 h (LiHep), and 4.3 h (serum)
- Lactic acid/glucose ratio stability falls in the reported 21–32 min range
- Every parameter × tube type × delay-type combination present in the source tables appears in results_table.csv with no missing rows
- Any recomputed time point diverging from its reported anchor value is flagged for expert review rather than silently accepted

## Limitations

- Stability time points are defined only for the 20% concentration-change threshold under room-temperature delays in the studied tube types (EDTA, LiHep, serum); other thresholds or temperatures are not covered
- Effects are metabolite-specific: some metabolites (e.g., lactic acid) are affected by even short delays while others remain stable longer, so a single global threshold is inappropriate
- No changelog or version information is provided for the PRIMA Panel repository, and no software version numbers are stated for the hosted Shiny app
- No explicit limitations of the models or the PRIMA Panel are discussed in the source context; robustness was only checked via a separate validation cohort

## Evidence

- [abstract] stability time point definition: "The study introduced the concept of a “stability time point”, marking when a metabolite’s concentration changes by 20%"
- [abstract] linear mixed model fitting: "fitted in linear mixed models used to predict changes in metabolite concentrations over time"
- [intro] model purpose: "These data were then used to generate linear mixed models that allow for prediction and quantification of the changes each metabolic parameter undergoes with prolonged processing delays"
- [readme] PRIMA Panel Data tab display of time points: "The Data tab shows different ways of highlighting the different stability time-points in minutes. The time-points are sorted according to their SPREC classification."
- [readme] performance report generation: "The 'Single' tab allows the user to set a pre-centrifugation time and a post-centrifugation using a Slider. A table is then generated that higlights a minor and a major change"
- [other] lactic acid anchor value: [evidence span withheld — anchor status: not_found]
