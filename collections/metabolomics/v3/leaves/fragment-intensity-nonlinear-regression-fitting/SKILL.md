---
name: fragment-intensity-nonlinear-regression-fitting
description: Use when building targeted lipidomics PRM/MRM assays that need optimal
  collision energies per fragment from CE-dependent intensity measurements on Thermo
  QExactive HF or Agilent QTOF instruments.
when_to_use_negative:
- CE-dependent measurements for the lipid class of interest are unavailable — LipidCreator
  CE models are currently exclusive to lipid mediators and expansion requires availability
  of more structurally diverse lipid standards
- The user needs a scoring algorithm to determine false discovery rates or to resolve
  coeluting/isobaric lipids and peak area integration — LipidCreator intentionally
  does not address these
- Target organisms are bacteria such as E. coli or archaea, where rare lipidome-wide
  studies and explicit lack of standards for acyl-PG, acyl-PE, and di-/tetraacyl cardiolipins
  limit standard availability
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: flipR
  role: Fits fragment-specific dissociation profiles using nonlinear regression with
    an extended log-normal kernel to the CE-dependent relative fragment intensity
    data
  repo: https://github.com/lifs-tools/lipidcreator
- name: minpack.lm
  role: Performs the nonlinear regression based on the Levenberg-Marquardt nonlinear
    least squares algorithm for each fragment's dissociation profile
  repo: https://rdrr.io/cran/minpack.lm/
- name: nls.multstart
  role: Performs a bounded grid search optimization over the kernel parameters to
    obtain robust starting values and global fits
  repo: https://rdrr.io/cran/nls.multstart/
- name: LipidCreator
  role: Hosts the CE optimization module; the instrument-specific configuration allows
    definition of a minimum collision energy above which the software reports model-based
    predicted values, and exports blib spectral libraries after activating the collision
    energy module
  repo: https://github.com/lifs-tools/lipidcreator
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:d815498f1f9e3b2636d0efb26edd081a410931c11e4dd3e13d670f4032e1c805
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidcreator
  tools:
  - LipidCreator
  - flipR
  - minpack.lm
  - nls.multstart
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# fragment-intensity-nonlinear-regression-fitting

## Summary

Fit fragment-specific nonlinear regression models (extended log-normal kernel) to collision-energy-dependent relative fragment intensities from QExactive HF and QTOF standards, enabling model-based collision energy prediction for targeted lipidomics transitions in LipidCreator.

## When to use

Use when building targeted lipidomics PRM/MRM assays that need optimal collision energies per fragment from CE-dependent intensity measurements on Thermo QExactive HF or Agilent QTOF instruments.

## When NOT to use

- CE-dependent measurements for the lipid class of interest are unavailable — LipidCreator CE models are currently exclusive to lipid mediators and expansion requires availability of more structurally diverse lipid standards
- The user needs a scoring algorithm to determine false discovery rates or to resolve coeluting/isobaric lipids and peak area integration — LipidCreator intentionally does not address these
- Target organisms are bacteria such as E. coli or archaea, where rare lipidome-wide studies and explicit lack of standards for acyl-PG, acyl-PE, and di-/tetraacyl cardiolipins limit standard availability

## Inputs

- Collision-energy-dependent relative fragment intensity measurements from CE-dependent calibration data covering QExactive HF NCE 10-60 and QTOF CE 10-100 ranges
- User-selected fragment transition lists for a targeted lipidomics assay

## Outputs

- Fragment-specific fitted nonlinear regression model parameters (extended log-normal kernel coefficients per fragment)
- Relative residuals for each fitted model
- Coverage statistics per instrument (fraction of models with relative residual ≤ 10%)
- Optimal collision energy per fragment (mode of the product density distribution across fragment distributions)

## How to apply

Load relative fragment intensities measured across an NCE 10-60 range on QExactive HF or a CE 10-100 range on QTOF. For each fragment, fit a nonlinear regression using the extended log-normal kernel implemented in the R package minpack.lm (Levenberg-Marquardt nonlinear least squares), performing a bounded grid search optimization with nls.multstart to obtain robust starting values and global fits. Compute residuals for each fitted model and convert them to relative residuals with respect to the measured intensities; the coverage statistic is the fraction of models whose relative residual is at most 10% (the paper reports 99.7% for QExactive HF models). Select the optimal CE as the mode of the product density distribution over all individual fragment distributions, i.e. the CE covering the highest simultaneous product overlap across the user's selected fragments.

## Related tools

- **flipR** (Fits fragment-specific dissociation profiles using nonlinear regression with an extended log-normal kernel to the CE-dependent relative fragment intensity data) — https://github.com/lifs-tools/lipidcreator
- **minpack.lm** (Performs the nonlinear regression based on the Levenberg-Marquardt nonlinear least squares algorithm for each fragment's dissociation profile) — https://rdrr.io/cran/minpack.lm/
- **nls.multstart** (Performs a bounded grid search optimization over the kernel parameters to obtain robust starting values and global fits) — https://rdrr.io/cran/nls.multstart/
- **LipidCreator** (Hosts the CE optimization module; the instrument-specific configuration allows definition of a minimum collision energy above which the software reports model-based predicted values, and exports blib spectral libraries after activating the collision energy module) — https://github.com/lifs-tools/lipidcreator

## Examples

```
Rscript -e "library(minpack.lm); library(nls.multstart); fit <- nlsLM(rel_int ~ flipR::lognormal_kernel(ce, mu, sigma, alpha), data = ce_data, start = list(mu = 20, sigma = 5, alpha = 1))"
```

## Evaluation signals

- Fraction of fitted models whose relative residual is at most 10% should match the paper's reported coverage (99.7% for QExactive HF models)
- Fitted model parameters and relative residuals must be written out to the ce_model_fit_report.csv report
- The optimal CE selected as the mode of the product density distribution must correspond to the CE covering the highest simultaneous product overlap across the selected fragments
- The exported blib file must contain the CE-optimized transitions when the collision energy module is activated in LipidCreator
- Instrument-specific minimum CE threshold must be respected: model-based predicted values are only reported above this configurable minimum

## Limitations

- LipidCreator CE models are currently exclusive to lipid mediators; expansion to other lipid classes requires availability of more structurally diverse lipid standards and CE-dependent measurements of their fragmentation behavior
- Due to the lack of available standards in rare lipidome-wide studies for bacteria such as E. coli and archaea, and the explicit lack of standards for acyl-PG, acyl-PE in E. coli and di-/tetraacyl cardiolipins, the current implementation focuses on eukaryotic model systems
- The majority of eukaryotic lipid classes are covered with some exceptions (Supplementary Data 1)

## Evidence

- [discussion] flipR uses nonlinear regression with an extended log-normal kernel to fit fragment-specific dissociation profiles: "flipR uses nonlinear regression with an extended log-normal kernel to fit fragment-specific dissociation profiles"
- [discussion] We use the R-packages minpack.lm (https://rdrr.io/cran/minpack.lm/), for nonlinear regression based on the Levenberg-Marquardt nonlinear least squares algorithm: "We use the R-packages minpack.lm (https://rdrr.io/cran/minpack.lm/), for nonlinear regression based on the Levenberg-Marquardt nonlinear least squares algorithm"
- [discussion] and nls.multstart (https://rdrr.io/cran/nls.multstart/) to perform a bounded grid search optimization: "and nls.multstart (https://rdrr.io/cran/nls.multstart/) to perform a bounded grid search optimization"
- [abstract] LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes.: "LipidCreator offers a comprehensive framework to compute MS/MS fragment masses for over 60 lipid classes."
- [abstract] LipidCreator provides all functionalities needed to define fragments, manage stable isotope labeling, optimize collision energy and generate in silico spectral libraries.: "LipidCreator provides all functionalities needed to define fragments, manage stable isotope labeling, optimize collision energy and generate in silico spectral libraries."
- [discussion] the instrument specific configuration in LipidCreator allows the definition of a minimum collision energy above which the software reports the model-based predicted values.: "the instrument specific configuration in LipidCreator allows the definition of a minimum collision energy above which the software reports the model-based predicted values."
- [methods] Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions: "Try to export a blib file (after activating the collision energy module) from the lipid review dialog after adding some transitions"
- [discussion] This requires the availability of more structurally diverse lipid standards and CE-dependent measurements of their fragmentation behavior: "This requires the availability of more structurally diverse lipid standards and CE-dependent measurements of their fragmentation behavior"
