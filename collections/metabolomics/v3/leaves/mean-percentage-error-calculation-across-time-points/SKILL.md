---
name: mean-percentage-error-calculation-across-time-points
description: 'Use when validating delay-prediction models for NMR metabolomics data:
  you have fitted linear mixed models of metabolite change over pre-/post-centrifugation
  delays and need to score prediction accuracy against measured time-point values.'
when_to_use_negative:
- The metabolite's concentration does not follow a linear response over the delay
  range (e.g., glucose at 6 and 8 h), where MPE will misleadingly spike — consider
  a non-linear or stratified approach instead.
- You have only the discovery cohort (no independent validation cohort); MPE computed
  on the training data would overstate model robustness.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_0091
tools:
- name: PRIMA Panel
  role: Open-source R Shiny tool from which the linear mixed models and validation
    data are drawn; the MPE calculation is part of its validation-cohort evaluation
    workflow.
  repo: https://github.com/funkam/PRIMA
- name: R
  role: Environment in which the linear mixed models are applied to validation-cohort
    time-0 values and the MPE table is written.
  repo: https://github.com/funkam/PRIMA
- name: NMR spectroscopy
  role: Provides the measured metabolite concentration values that MPE predictions
    are compared against.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:dda77c4311d71059039b3d7dec5952d8939a9d316c65c7eb63f6ac49d76aaee4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/funkam/PRIMA
  tools:
  - NMR spectroscopy
  - PRIMA Panel
  - R
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# mean-percentage-error-calculation-across-time-points

## Summary

Computes mean percentage error (MPE) between linear mixed-model predictions and measured NMR concentrations of blood-derived metabolites at each processing-delay time point, to quantify how robustly time-0 measurements predict later metabolite states.

## When to use

Use when validating delay-prediction models for NMR metabolomics data: you have fitted linear mixed models of metabolite change over pre-/post-centrifugation delays and need to score prediction accuracy against measured time-point values.

## When NOT to use

- The metabolite's concentration does not follow a linear response over the delay range (e.g., glucose at 6 and 8 h), where MPE will misleadingly spike — consider a non-linear or stratified approach instead.
- You have only the discovery cohort (no independent validation cohort); MPE computed on the training data would overstate model robustness.

## Inputs

- Fitted linear mixed models of metabolite concentration vs. processing delay
- Validation-cohort time-0 metabolite measurements (NMR-quantified, pre- and postcentrifugation delay groups)
- Measured NMR concentration values at 2, 4, 6, and 8 h time points for the validation cohort

## Outputs

- MPE values per metabolite per time point
- Validation_mpe_table.csv — metabolite-by-time-point MPE table

## How to apply

Load the fitted linear mixed models and time-0 measurements from the validation cohort (n = 10 precentrifugation, n = 7 postcentrifugation samples). For each metabolite in each delay category, apply the linear mixed model to predict concentrations at the 2, 4, 6, and 8 h time points from the time-0 value, then compare each prediction against the measured NMR value at that time point. Compute mean percentage error (MPE) per metabolite and time point as the mean of (predicted - measured)/measured relative deviations across the validation cohort. Interpret results against the paper's stated criterion of MPE well below 10% at the 2 and 4 h time points, and flag exceptions such as glucose at later time points where concentration deviates from the linear model response.

## Related tools

- **PRIMA Panel** (Open-source R Shiny tool from which the linear mixed models and validation data are drawn; the MPE calculation is part of its validation-cohort evaluation workflow.) — https://github.com/funkam/PRIMA
- **R** (Environment in which the linear mixed models are applied to validation-cohort time-0 values and the MPE table is written.) — https://github.com/funkam/PRIMA
- **NMR spectroscopy** (Provides the measured metabolite concentration values that MPE predictions are compared against.)

## Examples

```
Rscript -e 'pred <- predict(lmm, newdata=valid); mpe <- mean((pred - valid$conc)/valid$conc); write.csv(mpe, "validation_mpe_table.csv")'
```

## Evaluation signals

- Computed MPE at 2 and 4 h time points should fall well below 10% for most parameters, matching the reported validation pattern.
- Postcentrifugation acetone in EDTA plasma should surface as a known exception at the 2 and 4 h time points.
- Glucose in EDTA and LiHep plasma should show unexpectedly large MPE at 6 and 8 h, consistent with its non-linear response.
- Output CSV must contain one row per metabolite per time point with a numeric MPE column.

## Limitations

- MPE is unreliable for metabolites whose concentration does not follow the linear model response, as demonstrated by glucose at later time points in EDTA and LiHep plasma.
- The validation cohort is small (n = 10 precentrifugation, n = 7 postcentrifugation), so MPE estimates carry considerable sampling uncertainty.
- The PRIMA Panel itself does not version its models or document a changelog, so MPE reproducibility across panel releases is not guaranteed.

## Evidence

- [abstract] Validation cohort MPE below 10% at 2 and 4 h: "These predictive models were validated in a separate cohort"
- [intro] Glucose non-linear response breaks model accuracy: "These data were then used to generate linear mixed models that allow for prediction and quantification of the changes each metabolic parameter undergoes with prolonged processing delays"
- [discussion] PRIMA Panel performance reports combine models and data: "these models and data were combined to create the PRIMA Panel, which allows the exploration of the data and provides output in form of performance reports"
- [readme] PRIMA Panel online deployment and repository: "This Shiny app is also permantently hosted on Shinyapps.io"
