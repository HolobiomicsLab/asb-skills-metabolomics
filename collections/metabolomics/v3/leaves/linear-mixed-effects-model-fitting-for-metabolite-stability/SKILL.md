---
name: linear-mixed-effects-model-fitting-for-metabolite-stability
description: Use when NMR metabolomics concentrations are available for blood samples
  exposed to varying pre-/postcentrifugation delays across tube types and you must
  quantify how fast each metabolite changes (time to 20% concentration change).
when_to_use_negative:
- The goal is a pass/fail quality flag for an uploaded sample cohort — use the PRIMA
  Panel performance-report workflow instead of refitting models
- Samples were not collected in peripheral-blood tubes (plasma/serum) with recorded
  pre-/postcentrifugation delays, so delay effects cannot be modeled
- No repeated measures per donor exist, so a random-intercept structure (donor) is
  not identifiable
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3520
tools:
- name: R
  role: Environment for loading the delay-experiment tables and fitting the linear
    mixed models used to predict concentration changes over time
  repo: github.com/funkam/PRIMA
- name: PRIMA Panel
  role: Open-source R Shiny tool that packages the fitted models to explore stability
    time points and generate performance reports from uploaded pre-analytical tables
  repo: github.com/funkam/PRIMA
- name: PRIMA
  role: Online-hosted Shiny instance of the panel for interactive exploration of stability
    time points
  repo: https://funkam.shinyapps.io/PRIMA/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:2d56cb57a0760421f6695dd0f0dc17c78ba432858040442a8a0ab2e1570bbe9c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PRIMA
  - PRIMA Panel
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# linear-mixed-effects-model-fitting-for-metabolite-stability

## Summary

Fits linear mixed models to NMR-quantified metabolite concentrations measured after controlled pre- and postcentrifugation delays, with donor as random intercept and cell count as random slope, to derive each parameter's 20% 'stability time point' per tube type (EDTA plasma, LiHep plasma, serum).

## When to use

Use when NMR metabolomics concentrations are available for blood samples exposed to varying pre-/postcentrifugation delays across tube types and you must quantify how fast each metabolite changes (time to 20% concentration change).

## When NOT to use

- The goal is a pass/fail quality flag for an uploaded sample cohort — use the PRIMA Panel performance-report workflow instead of refitting models
- Samples were not collected in peripheral-blood tubes (plasma/serum) with recorded pre-/postcentrifugation delays, so delay effects cannot be modeled
- No repeated measures per donor exist, so a random-intercept structure (donor) is not identifiable

## Inputs

- Delay-experiment data tables from the PRIMA GitHub repository (github.com/funkam/PRIMA) loaded into R
- Per-metabolite concentration values from NMR spectroscopy with tube type (EDTA, LiHep, serum), pre-/postcentrifugation delay times, donor identifiers, and cell counts

## Outputs

- results_table.csv with recomputed stability time points per metabolic parameter, tube type, and delay type
- Fitted linear mixed models predicting concentration change over prolonged processing delays

## How to apply

Structure the delay-experiment data per metabolic parameter with tube type (EDTA, LiHep, serum), pre- and postcentrifugation delay times, donor identifiers, and cell counts. Fit a linear mixed model per parameter with cell count as random slope and donor as random intercept, so predicted concentration trajectories account for donor-level correlation and cellular contamination. From each fitted model, extract the modeled time at which concentration changes by 20% — the stability time point — separately for pre- and postcentrifugation delays. Validate against reported anchors, e.g., lactic acid precentrifugation 0.41 h in EDTA plasma (about 25 min at room temperature) and postcentrifugation 0.81 h (EDTA), 1.63 h (LiHep), 4.3 h (serum), plus the lactic acid/glucose ratio change of 21–32 min; flag divergent parameters for expert review and write results per parameter, tube type, and delay type to results_table.csv.

## Related tools

- **R** (Environment for loading the delay-experiment tables and fitting the linear mixed models used to predict concentration changes over time) — github.com/funkam/PRIMA
- **PRIMA Panel** (Open-source R Shiny tool that packages the fitted models to explore stability time points and generate performance reports from uploaded pre-analytical tables) — github.com/funkam/PRIMA
- **PRIMA** (Online-hosted Shiny instance of the panel for interactive exploration of stability time points) — https://funkam.shinyapps.io/PRIMA/

## Evaluation signals

- Recomputed lactic acid precentrifugation stability time point in EDTA plasma reproduces 0.41 h (~25 min) within the table's reported precision
- Recomputed lactic acid postcentrifugation stability time points match 0.81 h (EDTA), 1.63 h (LiHep), and 4.3 h (serum)
- Recomputed lactic acid/glucose ratio change falls in the reported 21–32 min range
- Model fits include donor as random intercept and cell count as random slope; stability time points are stratified by tube type and delay type (SPREC classification)
- Any parameter whose recomputed time point diverges from the reported value is flagged for expert review

## Limitations

- No version information or changelog is provided for the PRIMA Panel, and no software version numbers are stated in the methods
- Stability time points are model-based extrapolations from the delay experiment; validity depends on the linear mixed-model assumptions and on the validation cohort used to test model robustness

## Evidence

- [intro] These data were then used to generate linear mixed models that allow for prediction and quantification of the changes each metabolic parameter undergoes with prolonged processing delays: "These data were then used to generate linear mixed models that allow for prediction and quantification of the changes each metabolic parameter undergoes with prolonged processing delays"
- [abstract] The study introduced the concept of a 'stability time point', marking when a metabolite's concentration changes by 20%: "The study introduced the concept of a “stability time point”, marking when a metabolite’s concentration changes by 20%"
- [readme] The Data tab shows different ways of highlighting the different stability time-points in minutes. The time-points are sorted according to their SPREC classification.: "The Data tab shows different ways of highlighting the different stability time-points in minutes. The time-points are sorted according to their SPREC classification."
- [abstract] These predictive models were validated in a separate cohort: "These predictive models were validated in a separate cohort"
