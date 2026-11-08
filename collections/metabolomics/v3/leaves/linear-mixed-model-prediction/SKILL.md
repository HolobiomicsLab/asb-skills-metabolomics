---
name: linear-mixed-model-prediction
description: Use when NMR-quantified plasma/serum metabolite concentrations are available
  at multiple controlled pre- or postcentrifugation delay time points (e.g. 2, 4,
  6, 8 h) and per-parameter concentration changes must be predicted or a stability
  time point (20% change) must be located.
when_to_use_negative:
- The input is not peripheral-blood plasma/serum NMR data with known collection, centrifugation,
  and freezing timestamps — the models were trained only on this preanalytical design.
- The metabolite of interest does not follow the linear response captured by the models,
  e.g. glucose in EDTA and LiHep plasma, which showed unexpectedly large MPE at later
  (6 and 8 h) time points.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3272
- http://edamontology.org/topic_3360
tools:
- name: PRIMA Panel
  role: Open-source R Shiny tool that hosts the fitted linear mixed models, computes
    pre- and postcentrifugation delays from uploaded date-time stamps, predicts per-parameter
    concentration changes, and generates performance reports
  repo: https://funkam.shinyapps.io/PRIMA/
- name: R
  role: Language in which the PRIMA Panel and its linear mixed models are implemented
    and hosted
  repo: https://github.com/funkam/PRIMA
- name: NMR spectroscopy
  role: Quantifies metabolite levels that serve as the response variable for fitting
    and validating the linear mixed models
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:d59b045899a46fc218b772b68b1722ae46795041cf957eec90735497bcf59723
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NMR spectroscopy
  - PRIMA Panel
  - R
  - Shiny
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3272
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# linear-mixed-model-prediction

## Summary

Fits and applies linear mixed models to NMR-quantified metabolite concentrations measured across controlled pre- and postcentrifugation delays, so that concentration changes for each metabolic parameter can be predicted and quantified at arbitrary processing times.

## When to use

Use when NMR-quantified plasma/serum metabolite concentrations are available at multiple controlled pre- or postcentrifugation delay time points (e.g. 2, 4, 6, 8 h) and per-parameter concentration changes must be predicted or a stability time point (20% change) must be located.

## When NOT to use

- The input is not peripheral-blood plasma/serum NMR data with known collection, centrifugation, and freezing timestamps — the models were trained only on this preanalytical design.
- The metabolite of interest does not follow the linear response captured by the models, e.g. glucose in EDTA and LiHep plasma, which showed unexpectedly large MPE at later (6 and 8 h) time points.

## Inputs

- NMR-quantified metabolite concentration tables for blood collected in various tube types and subjected to controlled pre- and postcentrifugation delays
- Timestamps of blood collection, centrifugation, and freezing (date+time stamps) for computing pre- and postcentrifugation delays
- A separate validation cohort with time-0 measurements and measured NMR values at 2, 4, 6, and 8 h delay time points (n = 10 precentrifugation, n = 7 postcentrifugation)

## Outputs

- Fitted linear mixed models (one per metabolic parameter and delay type) for predicting concentration changes over time
- Predicted concentration-change values per parameter at chosen pre-/postcentrifugation delays, combined for a comprehensive overview
- Stability time points (the delay at which a metabolite's concentration changes by 20%)
- Validation mean-percentage-error (MPE) values per metabolite and time point

## How to apply

Fit one linear mixed model per metabolic parameter and per delay type (pre- vs postcentrifugation) using metabolite levels quantified by NMR spectroscopy across controlled delays, treating sample-level variation as a random effect so within- and between-sample variability are separated. Use the fitted models to predict concentration changes over time starting from the time-0 measurement, combining the pre- and postcentrifugation model outputs to obtain a comprehensive overview of a sample's preanalytical history. Define each parameter's 'stability time point' as the delay at which the predicted concentration changes by 20%. Validate the models by applying them to a separate validation cohort (n = 10 precentrifugation, n = 7 postcentrifugation) and computing the mean percentage error (MPE) of predicted vs measured NMR values at the 2, 4, 6, and 8 h time points; expect MPE well below 10% at 2 and 4 h for most parameters. Deploy the models through the PRIMA Panel R Shiny tool (github.com/funkam/PRIMA) so delays can be explored interactively and performance reports generated for sample cohorts.

## Related tools

- **PRIMA Panel** (Open-source R Shiny tool that hosts the fitted linear mixed models, computes pre- and postcentrifugation delays from uploaded date-time stamps, predicts per-parameter concentration changes, and generates performance reports) — https://funkam.shinyapps.io/PRIMA/
- **R** (Language in which the PRIMA Panel and its linear mixed models are implemented and hosted) — https://github.com/funkam/PRIMA
- **NMR spectroscopy** (Quantifies metabolite levels that serve as the response variable for fitting and validating the linear mixed models)

## Examples

```
shiny::runGitHub('funkam/PRIMA')  # launch PRIMA Panel; upload timestamp table to compute pre-/post-centrifugation delays and apply the deposited linear mixed models per parameter
```

## Evaluation signals

- Predicted vs measured MPE at the 2 and 4 h validation time points is well below 10% for most parameters
- Postcentrifugation acetone in EDTA plasma is a known exception with higher MPE at 2–4 h
- Glucose in EDTA and LiHep plasma shows unexpectedly large MPE at the 6 and 8 h time points because its concentration does not follow the linear response
- Each parameter's stability time point is reproducible as the predicted delay at which concentration changes by 20%
- Pre- and postcentrifugation delays recomputed from collection/centrifugation/freezing date+time stamps match the SPREC classification used to sort the stability time points

## Limitations

- Glucose in EDTA and LiHep plasma does not follow the linear response of the models, causing unexpectedly large MPE at later time points
- Postcentrifugation acetone in EDTA plasma also deviates from the expected MPE pattern at the 2–4 h time points
- Validation cohort size was small (n = 10 precentrifugation, n = 7 postcentrifugation), so model robustness was assessed on limited data
- No explicit version numbers or changelog are provided for the PRIMA Panel implementation of the models

## Evidence

- [abstract] Predictive models trained on delayed samples: "fitted in linear mixed models used to predict changes in metabolite concentrations over time"
- [abstract] Stability time point definition: "The study introduced the concept of a “stability time point”, marking when a metabolite’s concentration changes by 20%"
- [abstract] Validation cohort results: "These predictive models were validated in a separate cohort"
- [discussion] Combined modeling rationale: "a combination of pre- and postcentrifugation delays was modeled to obtain a comprehensive overview of preanalytical process for metabolomics samples"
- [discussion] Model validation procedure: "A validation cohort was used to investigate the robustness of these models"
- [readme] Tools tab delay computation: "a tool for calculating the pre- and post-centrifugation times from the differences of date+time stamps."
