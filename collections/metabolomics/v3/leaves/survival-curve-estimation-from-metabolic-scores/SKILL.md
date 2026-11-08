---
name: survival-curve-estimation-from-metabolic-scores
description: Use when the analysis goal is examining how a projected metabolomics-based
  score (e.g., mortality score) predicts events over time, and the dataset contains
  samples with both a projected score and event/time endpoint data.
when_to_use_negative:
- The dataset has no event/time endpoint columns (e.g., only metabolite measurements
  without mortality or disease follow-up) — score projection or correlation analysis
  should be used instead
- The goal is associating scores with metabolites or phenotypic variables (MetaboWAS/cor_assoc)
  rather than time-to-event prediction
- Working outside the MiMIR framework with non-Nightingale 1H-NMR metabolomics scores
  that lack the expected sample-to-score structure
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: MiMIR
  role: Provides the kapmeier_scores helper and the projected metabolic scores (e.g.,
    mortality score) used for Kaplan–Meier event-prediction analysis
  repo: https://github.com/DanieleBizzarri/MiMIR
- name: survival
  role: R package underlying kapmeier_scores for fitting survival curves over time
- name: survminer
  role: R package used to render the Kaplan–Meier figure from the fitted survival
    curves
- name: ggplot2
  role: Graphics library backing survminer plot rendering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btac388
    title: 'MiMIR: R-shiny application to infer risk factors and endpoints from Nightingale
      Health’s 1H-NMR metabolomics data'
  source_content_hash: sha256:99390495bee9ad0dc6963d5daccaf1c0c6281c3d5567cf31f78b201b4817a5bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DanieleBizzarri/MiMIR
  tools:
  - MiMIR
  - R
  - ggplot2
  - matrixStats
  - shiny
  - stats
  - survival
  - survminer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45
n_claims: 20
---

# survival-curve-estimation-from-metabolic-scores

## Summary

Estimates event-over-time prediction using Kaplan–Meier survival curves computed from MiMIR's projected metabolomics-based scores (e.g., the mortality score) via the kapmeier_scores helper, built on the R survival and survminer packages.

## When to use

Use when the analysis goal is examining how a projected metabolomics-based score (e.g., mortality score) predicts events over time, and the dataset contains samples with both a projected score and event/time endpoint data.

## When NOT to use

- The dataset has no event/time endpoint columns (e.g., only metabolite measurements without mortality or disease follow-up) — score projection or correlation analysis should be used instead
- The goal is associating scores with metabolites or phenotypic variables (MetaboWAS/cor_assoc) rather than time-to-event prediction
- Working outside the MiMIR framework with non-Nightingale 1H-NMR metabolomics scores that lack the expected sample-to-score structure

## Inputs

- Projected metabolic scores matrix/data frame (e.g., mortality score per sample) from MiMIR score projection
- Phenotypic dataset containing event indicators and follow-up times (e.g., MiMIR synthetic_phenotypic_dataset)
- Metabolites data uploaded as CSV/TSV with the same column names as the MiMIR example dataset

## Outputs

- Kaplan–Meier survival curve figure rendered with survminer/ggplot2 showing event prediction over time per score group
- Survival fit objects (survival::survfit) underlying the plots

## How to apply

First load the projected metabolic scores and phenotypic data (e.g., the synthetic_metabolic_dataset and synthetic_phenotypic_dataset shipped with MiMIR for testing) so that each sample has a score value plus an event indicator and follow-up time. Ensure the survival and survminer libraries are installed, as listed in MiMIR's requirements. Call the kapmeier_scores helper from MiMIR to fit Kaplan–Meier curves for the mortality score (typically stratifying samples into score-based groups), which wraps survival::survfit. Render the result with survminer (ggplot2-based) to produce the Kaplan–Meier figure showing event prediction over time. Correct application is judged by whether the rendered figure shows separated survival curves across score strata with confidence bands, and whether the curves reflect the endpoint columns supplied in the phenotypic data.

## Related tools

- **MiMIR** (Provides the kapmeier_scores helper and the projected metabolic scores (e.g., mortality score) used for Kaplan–Meier event-prediction analysis) — https://github.com/DanieleBizzarri/MiMIR
- **survival** (R package underlying kapmeier_scores for fitting survival curves over time)
- **survminer** (R package used to render the Kaplan–Meier figure from the fitted survival curves)
- **ggplot2** (Graphics library backing survminer plot rendering)

## Examples

```
if (!require("survival")) install.packages("survival"); if (!require("survminer")) install.packages("survminer")
```

## Evaluation signals

- The kapmeier_scores call runs without error and returns fitted curves for the mortality score strata
- The rendered survminer figure displays distinct Kaplan–Meier curves (with confidence bands) per score group over follow-up time
- Event counts and curve steps match the event/time columns supplied in the phenotypic dataset (verifiable against the synthetic_phenotypic_dataset)
- All required R packages (survival, survminer, ggplot2) are present, satisfying the README's manual-install requirement

## Limitations

- Requires samples to have both projected scores and event/time endpoints; metabolomics data alone is insufficient
- Curves are univariate group summaries, not adjusted multivariate hazard models
- Relies on MiMIR's expected column naming, so datasets must match the example dataset's variable names or QC/alignment steps will fail

## Evidence

- [readme] README package requirements: "if (!require("survival")) install.packages("survival")
if (!require("survminer")) install.packages("survminer")"
- [results] kaplan-meier workflow in MiMIR: [evidence span withheld — anchor status: not_found]
- [methods] synthetic test datasets: "the metabolic synthetic dataset, the phenotypic synthetic dataset. These example dataset can be used to test the App"
- [abstract] score projection scope: "allows for the projection and calibration of 24 pre-trained metabolomics-based models, without any pre-required programming knowledge"
