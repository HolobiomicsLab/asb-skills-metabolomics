---
name: log-normal-kernel-dissociation-profile-modeling
description: Use when building targeted lipidomics assays that need per-fragment collision
  energy optimization from CE-dependent relative intensity measurements on QExactive
  HF or QTOF instruments.
when_to_use_negative:
- The target lipid class has no CE-dependent measurements from structurally diverse
  lipid standards — the paper states LipidCreator's CE models are currently exclusive
  to lipid mediators and expansion requires more CE-dependent measurements of fragmentation
  behavior.
- The analysis goal is transition-list validation or false-discovery-rate scoring
  rather than CE selection — LipidCreator intentionally does not address coeluting/isobaric
  lipids or peak area integration.
- Instruments other than the calibrated Thermo QExactive HF or Agilent QTOF, since
  no fitted dissociation profiles exist for other platform CE ranges.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3172
tools:
- name: flipR
  role: Fits fragment-specific dissociation profiles via nonlinear regression with
    an extended log-normal kernel
- name: minpack.lm
  role: Provides nonlinear regression based on the Levenberg-Marquardt nonlinear least
    squares algorithm used to fit the kernel
  repo: https://rdrr.io/cran/minpack.lm/
- name: nls.multstart
  role: Performs bounded grid search optimization over kernel parameters for robust
    starting values and global fits
  repo: https://rdrr.io/cran/nls.multstart/
- name: LipidCreator
  role: Hosts the collision energy optimization module and applies the fitted models
    to select optimal CEs for targeted assays
  repo: https://github.com/lifs-tools/lipidcreator
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:0dd1fd873641a6acdbe52f4442a46cc9b8f0312664c2a13632ccfeb6eb23106e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LipidCreator
  - flipR
  - minpack.lm
  - nls.multstart
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# log-normal-kernel-dissociation-profile-modeling

## Summary

Fits fragment-specific nonlinear regression models using an extended log-normal kernel to CE-dependent relative fragment intensities (QExactive HF NCE 10–60; QTOF CE 10–100) so the optimal collision energy for a targeted lipidomics assay can be selected as the mode of the product density distribution over all fragment profiles.

## When to use

Use when building targeted lipidomics assays that need per-fragment collision energy optimization from CE-dependent relative intensity measurements on QExactive HF or QTOF instruments.

## When NOT to use

- The target lipid class has no CE-dependent measurements from structurally diverse lipid standards — the paper states LipidCreator's CE models are currently exclusive to lipid mediators and expansion requires more CE-dependent measurements of fragmentation behavior.
- The analysis goal is transition-list validation or false-discovery-rate scoring rather than CE selection — LipidCreator intentionally does not address coeluting/isobaric lipids or peak area integration.
- Instruments other than the calibrated Thermo QExactive HF or Agilent QTOF, since no fitted dissociation profiles exist for other platform CE ranges.

## Inputs

- CE-dependent relative fragment intensity measurements over QExactive HF NCE 10–60
- CE-dependent relative fragment intensity measurements over QTOF CE 10–100
- User-selected fragment list for a targeted lipidomics transition set

## Outputs

- Fragment-specific fitted log-normal kernel parameters per instrument
- Relative residuals per fitted model
- Per-instrument coverage statistic (fraction of models with relative residual ≤ 10%)
- ce_model_fit_report.csv containing fitted parameters, relative residuals, and coverage statistics
- Optimal collision energy per fragment set (mode of the product density distribution)

## How to apply

Load collision-energy-dependent relative fragment intensities covering QExactive HF NCE 10–60 and QTOF CE 10–100, then fit a separate dissociation profile per fragment using flipR's extended log-normal kernel via the R package minpack.lm, which implements Levenberg-Marquardt nonlinear least squares. Use nls.multstart to perform a bounded grid search optimization over the kernel parameters, obtaining robust starting values and global fits, repeating the procedure separately per instrument. Convert fit residuals to relative residuals with respect to the measured intensities and evaluate the fraction of models whose relative residual is at most 10% per instrument (the paper reports 99.7% of QExactive HF models fall within this threshold). The optimal CE is then selected as the mode of the product density distribution over all individual fragment distributions — i.e. the collision energy covering the highest simultaneous product overlap across the user's selected fragments. Note that LipidCreator's instrument-specific configuration allows defining a minimum collision energy above which model-based predicted values are reported.

## Related tools

- **flipR** (Fits fragment-specific dissociation profiles via nonlinear regression with an extended log-normal kernel)
- **minpack.lm** (Provides nonlinear regression based on the Levenberg-Marquardt nonlinear least squares algorithm used to fit the kernel) — https://rdrr.io/cran/minpack.lm/
- **nls.multstart** (Performs bounded grid search optimization over kernel parameters for robust starting values and global fits) — https://rdrr.io/cran/nls.multstart/
- **LipidCreator** (Hosts the collision energy optimization module and applies the fitted models to select optimal CEs for targeted assays) — https://github.com/lifs-tools/lipidcreator

## Examples

```
fit <- nls.multstart::nls_multstart(intensity ~ flipR::lognormal_kernel(ce, a, mu, sigma), data = frag_ce, start = list(a = 1, mu = 25, sigma = 5), iter = 1000)
```

## Evaluation signals

- Relative residual threshold: at least ~99.7% of QExactive HF fitted models should show relative residuals ≤ 10% of measured intensities
- ce_model_fit_report.csv is produced and contains fitted parameters, relative residuals, and coverage statistics for both instruments
- The selected optimal CE equals the mode of the product density distribution over all individual fragment distributions, not any single fragment's peak CE
- Fits are obtained for both instrument ranges independently (QExactive HF NCE 10–60 and QTOF CE 10–100), so per-instrument parameter sets must be present
- A minimum collision energy boundary is defined in LipidCreator's instrument-specific configuration before model-based values are reported

## Limitations

- CE models are currently exclusive to lipid mediators; extending to other lipid classes requires more structurally diverse lipid standards and CE-dependent measurements of their fragmentation behavior
- Model quality depends on available standards — due to lack of standards for bacteria such as E. coli and archaea, coverage is limited to eukaryotic model systems
- The extended log-normal kernel fit quality must be verified per fragment via relative residuals; poor fits for a fragment degrade the simultaneous product-overlap CE selection

## Evidence

- [discussion] flipR nonlinear regression kernel: "Nonlinear Regression Model: flipR uses nonlinear regression with an extended log-normal kernel to fit fragment-specific dissociation profiles"
- [discussion] minpack.lm Levenberg-Marquardt: "We use the R-packages minpack.lm (https://rdrr.io/cran/minpack.lm/), for nonlinear regression based on the Levenberg-Marquardt nonlinear least squares algorithm)"
- [discussion] nls.multstart bounded grid search: "and nls.multstart (https://rdrr.io/cran/nls.multstart/) to perform a bounded grid search optimization"
- [discussion] minimum collision energy configuration: "the instrument specific configuration in LipidCreator allows the definition of a minimum collision energy above which the software reports the model-based predicted values."
- [readme] instrument support in README: "It has been tested with Thermo QExactive HF and Agilent QTOF instruments."
