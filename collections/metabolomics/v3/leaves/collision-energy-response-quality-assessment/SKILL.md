---
name: collision-energy-response-quality-assessment
description: Use when fitted flipR log-normal dissociation profiles over QExactive
  HF NCE 10-60 or QTOF CE 10-100 ranges need a fit-quality audit (relative residuals,
  coverage statistics) before selecting optimal CEs for a targeted lipidomics assay.
when_to_use_negative:
- The lipid class of interest lacks CE-dependent measurements from structurally diverse
  lipid standards, so no fitted dissociation profiles exist to audit
- You need a spectral library or transition list without collision-energy modeling
  — use LipidCreator's blib export directly instead of the CE fit-quality pipeline
- The data are DIA or full-scan peak areas rather than CE-dependent fragment intensity
  curves from PRM/SRM calibration standards
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: flipR
  role: Nonlinear regression with an extended log-normal kernel to fit fragment-specific
    dissociation profiles whose residuals and coverage are assessed
  repo: https://github.com/lifs-tools/lipidcreator
- name: minpack.lm
  role: Nonlinear regression based on the Levenberg-Marquardt nonlinear least squares
    algorithm used to fit each fragment's CE-response curve
  repo: https://rdrr.io/cran/minpack.lm/
- name: nls.multstart
  role: Bounded grid search optimization to obtain robust starting values and global
    fits before residual evaluation
  repo: https://rdrr.io/cran/nls.multstart/
- name: LipidCreator
  role: Hosts the CE optimization module, defines the instrument-specific minimum
    collision energy above which model-based predicted values are reported
  repo: https://github.com/lifs-tools/lipidcreator
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15960-z
    title: LipidCreator workbench to probe the lipidomic landscape
  source_content_hash: sha256:9f8a13e95a0a3c6870abb43cb6c2027595c81dd6b494b650fbf52acd73b84c36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lifs-tools/lipidcreator
  tools:
  - LipidCreator
  - flipR
  - minpack.lm
  - nls.multstart
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# collision-energy-response-quality-assessment

## Summary

Assesses the quality of fragment-specific nonlinear regression models fitted to CE-dependent relative fragment intensities by computing relative residuals and reporting the fraction of fits within a 10% residual threshold (e.g. 99.7% coverage on QExactive HF data), before trusting model-based collision-energy predictions.

## When to use

Use when fitted flipR log-normal dissociation profiles over QExactive HF NCE 10-60 or QTOF CE 10-100 ranges need a fit-quality audit (relative residuals, coverage statistics) before selecting optimal CEs for a targeted lipidomics assay.

## When NOT to use

- The lipid class of interest lacks CE-dependent measurements from structurally diverse lipid standards, so no fitted dissociation profiles exist to audit
- You need a spectral library or transition list without collision-energy modeling — use LipidCreator's blib export directly instead of the CE fit-quality pipeline
- The data are DIA or full-scan peak areas rather than CE-dependent fragment intensity curves from PRM/SRM calibration standards

## Inputs

- CE-dependent relative fragment intensity measurements from calibration standards (QExactive HF NCE 10-60 and QTOF CE 10-100 ranges)
- Fitted flipR nonlinear regression model parameters per fragment (extended log-normal kernel)
- LipidCreator instrument-specific configuration including the minimum collision energy threshold for model-based predictions

## Outputs

- ce_model_fit_report.csv containing fitted parameters, relative residuals, and coverage statistics per instrument
- Per-instrument coverage statistic (fraction of fragment models with relative residual <= 10%)
- Quality-flagged fragment dissociation profiles suitable for mode-of-product-density optimal CE selection

## How to apply

For each fragment, fit the extended log-normal kernel using the R package minpack.lm (Levenberg-Marquardt nonlinear least squares) with bounded grid-search starting values from nls.multstart, separately for the QExactive HF NCE 10-60 and QTOF CE 10-100 calibration datasets. Compute residuals for each fitted model and convert them to relative residuals with respect to the measured intensities. Evaluate the fraction of fitted models whose relative residual is at most 10% and record this coverage statistic per instrument; the paper reports 99.7% of QExactive HF models fall within this threshold. Only accept instrument configurations where coverage is high enough to trust model-based predicted values, noting LipidCreator's instrument-specific configuration allows defining a minimum collision energy above which model-based predicted values are reported. Write all fitted parameters, relative residuals, and coverage statistics to ce_model_fit_report.csv.

## Related tools

- **flipR** (Nonlinear regression with an extended log-normal kernel to fit fragment-specific dissociation profiles whose residuals and coverage are assessed) — https://github.com/lifs-tools/lipidcreator
- **minpack.lm** (Nonlinear regression based on the Levenberg-Marquardt nonlinear least squares algorithm used to fit each fragment's CE-response curve) — https://rdrr.io/cran/minpack.lm/
- **nls.multstart** (Bounded grid search optimization to obtain robust starting values and global fits before residual evaluation) — https://rdrr.io/cran/nls.multstart/
- **LipidCreator** (Hosts the CE optimization module, defines the instrument-specific minimum collision energy above which model-based predicted values are reported) — https://github.com/lifs-tools/lipidcreator

## Evaluation signals

- ce_model_fit_report.csv exists and contains per-fragment fitted parameters, relative residuals, and per-instrument coverage statistics
- Coverage statistic: fraction of fitted models with relative residual <= 10% matches the expected magnitude (99.7% on QExactive HF calibration data)
- Residuals are reported as relative residuals with respect to the measured intensities, not raw absolute residuals
- Separate coverage statistics are recorded for the QExactive HF NCE 10-60 and QTOF CE 10-100 datasets
- The chosen minimum collision energy in the LipidCreator instrument configuration is consistent with the range over which model-based predictions are trustworthy

## Limitations

- LipidCreator CE models are currently exclusive to lipid mediators; expansion to other lipid classes requires structurally diverse lipid standards and CE-dependent measurements of their fragmentation behavior
- Coverage depends on the quality of the underlying calibration data — models outside the 10% relative-residual threshold indicate fragment dissociation behavior poorly captured by the log-normal kernel
- Model-based predicted values are only reported above an instrument-specific minimum collision energy, so fits near or below that threshold cannot be trusted

## Evidence

- [discussion] flipR uses nonlinear regression with an extended log-normal kernel to fit fragment-specific dissociation profiles: "Nonlinear Regression Model: flipR uses nonlinear regression with an extended log-normal kernel to fit fragment-specific dissociation profiles"
- [discussion] minpack.lm Levenberg-Marquardt: "We use the R-packages minpack.lm (https://rdrr.io/cran/minpack.lm/), for nonlinear regression based on the Levenberg-Marquardt nonlinear least squares algorithm)"
- [discussion] nls.multstart bounded grid search: "and nls.multstart (https://rdrr.io/cran/nls.multstart/) to perform a bounded grid search optimization"
- [discussion] minimum collision energy above which model-based values are reported: "the instrument specific configuration in LipidCreator allows the definition of a minimum collision energy above which the software reports the model-based predicted values."
- [discussion] CE models exclusive to lipid mediators: "This requires the availability of more structurally diverse lipid standards and CE-dependent measurements of their fragmentation behavior"
