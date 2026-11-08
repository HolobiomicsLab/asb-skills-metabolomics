---
name: bounded-grid-search-parameter-estimation
description: Use when you must fit nonlinear (e.g. extended log-normal kernel) models
  to CE-dependent fragment intensities and need robust global parameter estimates
  via bounded grid search plus Levenberg-Marquardt refinement.
when_to_use_negative:
- No CE-dependent fragmentation measurements exist for the target lipid class — LipidCreator's
  CE models are currently exclusive to lipid mediators and expansion requires more
  structurally diverse lipid standards and CE-dependent measurements of their fragmentation
  behavior
- The data lack a swept independent variable (e.g. collision energy) — the bounded
  grid search over kernel parameters presumes CE-dependent relative intensities to
  fit dissociation profiles against
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: flipR
  role: Nonlinear regression with an extended log-normal kernel to fit fragment-specific
    dissociation profiles over the CE range
  repo: github.com/lifs-tools/lipidcreator
- name: minpack.lm
  role: Nonlinear regression based on the Levenberg-Marquardt nonlinear least squares
    algorithm for kernel fitting
  repo: https://rdrr.io/cran/minpack.lm/
- name: nls.multstart
  role: Bounded grid search optimization over kernel parameters to obtain robust starting
    values and global fits
  repo: https://rdrr.io/cran/nls.multstart/
- name: LipidCreator
  role: Hosts the CE optimization module; its instrument-specific configuration defines
    the minimum CE above which model-based predicted values are reported
  repo: https://github.com/lifs-tools/lipidcreator
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:3761e73c8734b55e17e495e81c623b22ba71c0ab57beae377c2050facbc5d12a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidCreator
  - flipR
  - minpack.lm
  - nls.multstart
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# bounded-grid-search-parameter-estimation

## Summary

Fits fragment-specific nonlinear regression models (an extended log-normal kernel) to collision-energy-dependent relative fragment intensities using minpack.lm's Levenberg-Marquardt least squares, stabilized by nls.multstart's bounded grid search for robust starting values and global fits. Used by LipidCreator/flipR to model dissociation profiles on QExactive HF (NCE 10–60) and QTOF (CE 10–100) instruments.

## When to use

Use when you must fit nonlinear (e.g. extended log-normal kernel) models to CE-dependent fragment intensities and need robust global parameter estimates via bounded grid search plus Levenberg-Marquardt refinement.

## When NOT to use

- No CE-dependent fragmentation measurements exist for the target lipid class — LipidCreator's CE models are currently exclusive to lipid mediators and expansion requires more structurally diverse lipid standards and CE-dependent measurements of their fragmentation behavior
- The data lack a swept independent variable (e.g. collision energy) — the bounded grid search over kernel parameters presumes CE-dependent relative intensities to fit dissociation profiles against

## Inputs

- Collision-energy-dependent relative fragment intensity measurements (QExactive HF NCE 10–60, QTOF CE 10–100)
- User-selected fragment list for a targeted lipidomics assay
- Instrument-specific configuration defining the minimum collision energy above which model-based predicted values are reported

## Outputs

- Per-fragment fitted kernel parameters and relative residuals per instrument
- ce_model_fit_report.csv with fitted parameters, relative residuals, and per-instrument coverage statistics
- Optimal CE selected as the mode of the product density distribution over all fragment distributions

## How to apply

Load the collision-energy-dependent relative fragment intensity measurements covering QExactive HF NCE 10–60 and QTOF CE 10–100 ranges, then, for each fragment, fit the extended log-normal kernel with the R package minpack.lm (Levenberg-Marquardt nonlinear least squares). Perform a bounded grid search optimization over the kernel parameters with nls.multstart to obtain robust starting values and global fits rather than trusting a single local optimum. Repeat the fit separately per instrument to obtain fragment-specific dissociation profiles. Compute residuals for each fitted model and convert them to relative residuals with respect to the measured intensities, then record the fraction of models whose relative residual is at most 10% as the coverage statistic per instrument. Write all fitted parameters, relative residuals, and coverage statistics to ce_model_fit_report.csv.

## Related tools

- **flipR** (Nonlinear regression with an extended log-normal kernel to fit fragment-specific dissociation profiles over the CE range) — github.com/lifs-tools/lipidcreator
- **minpack.lm** (Nonlinear regression based on the Levenberg-Marquardt nonlinear least squares algorithm for kernel fitting) — https://rdrr.io/cran/minpack.lm/
- **nls.multstart** (Bounded grid search optimization over kernel parameters to obtain robust starting values and global fits) — https://rdrr.io/cran/nls.multstart/
- **LipidCreator** (Hosts the CE optimization module; its instrument-specific configuration defines the minimum CE above which model-based predicted values are reported) — https://github.com/lifs-tools/lipidcreator

## Examples

```
library(nls.multstart); fit <- nls_multstart(y ~ exp(-0.5*((log(x)-mu)/sigma)^2), data=frag, start_lower=c(mu=10,sigma=0.1), start_upper=c(mu=60,sigma=5), iter=1000)
```

## Evaluation signals

- 99.7% of QExactive HF fitted models fall within the relative residual ≤ 10% threshold; coverage statistic per instrument should approach this
- ce_model_fit_report.csv exists and contains fitted parameters, relative residuals, and coverage statistics for every fragment–instrument pair
- Fits obtained via nls.multstart's bounded grid search converge to global rather than local minima of the Levenberg-Marquardt least-squares objective
- Resulting optimal CE equals the mode of the product density distribution, i.e. the CE covering the highest simultaneous product overlap across all selected fragments

## Limitations

- CE models are currently exclusive to lipid mediators; extension to other lipid classes requires more structurally diverse lipid standards and CE-dependent measurements of their fragmentation behavior
- Model-based predicted values are only reported above an instrument-specific configurable minimum collision energy

## Evidence

- [discussion] extended log-normal kernel fitting: "Nonlinear Regression Model: flipR uses nonlinear regression with an extended log-normal kernel to fit fragment-specific dissociation profiles"
- [discussion] Levenberg-Marquardt regression: "We use the R-packages minpack.lm (https://rdrr.io/cran/minpack.lm/), for nonlinear regression based on the Levenberg-Marquardt nonlinear least squares algorithm)"
- [discussion] bounded grid search: "and nls.multstart (https://rdrr.io/cran/nls.multstart/) to perform a bounded grid search optimization"
- [discussion] minimum CE reporting threshold: "the instrument specific configuration in LipidCreator allows the definition of a minimum collision energy above which the software reports the model-based predicted values."
- [readme] CE module activation check: "Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions"
