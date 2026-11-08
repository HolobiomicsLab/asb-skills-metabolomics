---
name: nmr-metabolomics-validation-cohort-analysis
description: Use when NMR metabolomics samples have known pre- or postcentrifugation
  delays and you need to test whether delay-prediction models (fit on a training cohort)
  robustly predict concentrations in an independent validation cohort, with a 10%
  MPE accuracy threshold.
when_to_use_negative:
- The goal is interactive exploration of stability time points or generating performance
  reports for a new sample cohort (use the PRIMA Panel Shiny app's Data/Single tabs
  instead of the validation workflow)
- The metabolite of interest is known to depart from the linear mixed-model response,
  e.g. glucose in EDTA or LiHep plasma at late time points, where predictions are
  unreliable
- No independent validation cohort or time-0 baseline measurement exists — the MPE
  calculation requires both
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PRIMA Panel
  role: Source of the deposited linear mixed models and data used for the validation-cohort
    evaluation; also available as an online Shiny app for interactive exploration
  repo: https://github.com/funkam/PRIMA
- name: R
  role: Language in which the linear mixed models are implemented and in which the
    validation MPE table is computed and exported
- name: NMR spectroscopy
  role: Quantification platform producing the measured metabolite concentrations against
    which model predictions are compared
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1021/acs.analchem.4c04938
    title: Validating Centralized Biobanking Workflows for NMR Metabolomics Using
      the PRIMA Panel
  source_content_hash: sha256:5e05149a7b7f389ca483c2506cd78e01c8c493b4a3626f626995816526ebf17c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/funkam/PRIMA
  tools:
  - NMR spectroscopy
  - PRIMA Panel
  - R
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# nmr-metabolomics-validation-cohort-analysis

## Summary

Reproduces the PRIMA Panel validation-cohort analysis: applying deposited linear mixed models to predict NMR metabolite concentrations at 2-, 4-, 6-, and 8-h processing delays from time-0 measurements, and quantifying agreement via mean percentage error (MPE) per metabolite and time point in a held-out cohort.

## When to use

Use when NMR metabolomics samples have known pre- or postcentrifugation delays and you need to test whether delay-prediction models (fit on a training cohort) robustly predict concentrations in an independent validation cohort, with a 10% MPE accuracy threshold.

## When NOT to use

- The goal is interactive exploration of stability time points or generating performance reports for a new sample cohort (use the PRIMA Panel Shiny app's Data/Single tabs instead of the validation workflow)
- The metabolite of interest is known to depart from the linear mixed-model response, e.g. glucose in EDTA or LiHep plasma at late time points, where predictions are unreliable
- No independent validation cohort or time-0 baseline measurement exists — the MPE calculation requires both

## Inputs

- Linear mixed models of metabolite concentration vs. processing delay deposited in the PRIMA repository (github.com/funkam/PRIMA)
- Validation-cohort time-0 NMR measurements split by precentrifugation (n = 10) and postcentrifugation (n = 7) delays
- Measured NMR metabolite concentrations at 2, 4, 6, and 8 h time points for the validation cohort

## Outputs

- Metabolite-by-time-point mean percentage error (MPE) table written to validation_mpe_table.csv
- Flags for metabolite/time-point combinations exceeding the 10% MPE threshold

## How to apply

Load the deposited linear mixed models and validation-cohort time-0 measurements from the PRIMA repository (github.com/funkam/PRIMA), splitting the cohort by delay type: n = 10 precentrifugation and n = 7 postcentrifugation samples. For each metabolite in each delay category, apply the linear mixed model to predict concentrations at the 2, 4, 6, and 8 h time points starting from the time-0 NMR measurement. Compare each prediction against the measured NMR value at the same time point and compute the mean percentage error (MPE) per metabolite and time point in R. Use 10% MPE as the decision threshold: at 2 and 4 h, MPE is expected to be well below 10% for most parameters, with postcentrifugation acetone in EDTA plasma as the known exception. Write the metabolite-by-time-point MPE table to validation_mpe_table.csv.

## Related tools

- **PRIMA Panel** (Source of the deposited linear mixed models and data used for the validation-cohort evaluation; also available as an online Shiny app for interactive exploration) — https://github.com/funkam/PRIMA
- **R** (Language in which the linear mixed models are implemented and in which the validation MPE table is computed and exported)
- **NMR spectroscopy** (Quantification platform producing the measured metabolite concentrations against which model predictions are compared)

## Examples

```
Rscript -e 'mpe_table <- compute_mpe(models, validation_cohort, time_points=c(2,4,6,8)); write.csv(mpe_table, "validation_mpe_table.csv")'
```

## Evaluation signals

- MPE is well below 10% for most parameters at the 2- and 4-h time points in both delay categories
- The known exception reproduces: postcentrifugation acetone in EDTA plasma shows elevated MPE at 2 and 4 h
- Glucose in EDTA and LiHep plasma shows unexpectedly large MPE at the late (6 and 8 h) time points because its concentration does not follow the linear response of the models
- The validation split is preserved exactly: n = 10 precentrifugation and n = 7 postcentrifugation samples
- validation_mpe_table.csv contains one row per metabolite per time point with an interpretable MPE value for each

## Limitations

- Prediction accuracy breaks down at late time points (6 and 8 h) for glucose in EDTA and LiHep plasma because its concentration does not follow the linear mixed-model response
- Postcentrifugation acetone in EDTA plasma fails the sub-10% MPE criterion even at short delays
- The PRIMA repository provides no changelog or version information, so reproducibility depends on pinning the retrieved model/data state manually

## Evidence

- [intro] linear mixed models generate delay predictions: "These data were then used to generate linear mixed models that allow for prediction and quantification of the changes each metabolic parameter undergoes with prolonged processing delays"
- [abstract] separate validation cohort: "These predictive models were validated in a separate cohort"
- [discussion] validation cohort tests model robustness: "A validation cohort was used to investigate the robustness of these models"
- [abstract] metabolites differ in delay sensitivity: "some metabolites, such as lactic acid, were significantly affected by even short delays, while others remained stable for longer"
- [readme] PRIMA Panel interactive delay exploration: "The 'Single' tab allows the user to set a pre-centrifugation time and a post-centrifugation using a Slider. A table is then generated that higlights a minor and a major change for each metabolite in"
