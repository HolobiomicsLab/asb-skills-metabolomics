---
name: synthetic-nmr-spectrum-simulation
description: Use when you must validate that an NMR fitting implementation reproduces
  documented lineshape equations — e.g. testing MultiNMRFit's mixed Gaussian-Lorentzian
  singlet, doublet, triplet, doublet-of-doublets, or quadruplet models against ground
  truth.
when_to_use_negative:
- Input is real experimental 1H/13C/31P spectra with unknown true parameter values
  — this skill targets synthetic validation against predefined ground truth, not routine
  spectrum fitting
- Analysis goal is pseudo-2D series fitting or biological interpretation rather than
  verifying that signal-model equations are correctly reconstructed
- Spectra require user-defined custom signal models outside the five built-in multiplicities
  (singlet through quadruplet)
tools:
- name: MultiNMRFit
  role: Fitting engine whose built-in multiplet signal models are validated against
    synthetic spectra with known ground truth
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
- name: Python
  role: Language used to implement the mixed Gaussian-Lorentzian lineshape functions
    and generate simulated 1D spectra (requires Python 3.8+)
- name: pip
  role: Installation channel for the multinmrfit package in the execution environment
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:0d0bd99f248c3bc4849d3a57be75518abc2b30e3362dcc95da5340c40f8e061f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNMRFit
  tools:
  - MultiNMRFit
  - Python
  - pip
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# synthetic-nmr-spectrum-simulation

## Summary

Simulate synthetic 1D NMR spectra with known ground-truth multiplet parameters and validate that signal-model fitting software (MultiNMRFit) recovers them, by implementing the built-in mixed Gaussian-Lorentzian singlet-to-quadruplet models and quantifying per-parameter recovery error.

## When to use

Use when you must validate that an NMR fitting implementation reproduces documented lineshape equations — e.g. testing MultiNMRFit's mixed Gaussian-Lorentzian singlet, doublet, triplet, doublet-of-doublets, or quadruplet models against ground truth.

## When NOT to use

- Input is real experimental 1H/13C/31P spectra with unknown true parameter values — this skill targets synthetic validation against predefined ground truth, not routine spectrum fitting
- Analysis goal is pseudo-2D series fitting or biological interpretation rather than verifying that signal-model equations are correctly reconstructed
- Spectra require user-defined custom signal models outside the five built-in multiplicities (singlet through quadruplet)

## Inputs

- Ground-truth parameter sets (xs, I, J, lw, a) for singlet, doublet, triplet, doublet-of-doublets, and quadruplet models
- Installed multinmrfit package (Python 3.8+)
- Synthetic 1D NMR spectra generated from the mixed Gaussian-Lorentzian model equations

## Outputs

- model_validation.csv — comparison table of fitted vs. simulated ground-truth values with recovery error per parameter and per model
- Fitted parameter values with estimated standard deviations for each multiplicity model

## How to apply

Install the multinmrfit package (Python 3.8+) via pip, then implement the five multiplet signal models as mixed Gaussian-Lorentzian lineshape functions parameterized by chemical shift xs, total intensity I, scalar coupling constant J, linewidth at half maximum lw, and Gaussian fraction a. Simulate synthetic 1D spectra using these models with predefined ground-truth values of xs, I, J, lw, and a for each multiplicity. Fit each simulated spectrum with MultiNMRFit's signal models and record fitted parameters alongside the ground-truth values. Compute recovery error per parameter (intensity, area, chemical shift, linewidth, coupling constant) for each model. The rationale is that MultiNMRFit extracts quantitative information (chemical shifts, signal intensity, coupling constants) by fitting series of 1D spectra, so closed-loop simulation with known truth is the direct test of implementation fidelity.

## Related tools

- **MultiNMRFit** (Fitting engine whose built-in multiplet signal models are validated against synthetic spectra with known ground truth) — https://github.com/NMRTeamTBI/MultiNMRFit/
- **Python** (Language used to implement the mixed Gaussian-Lorentzian lineshape functions and generate simulated 1D spectra (requires Python 3.8+))
- **pip** (Installation channel for the multinmrfit package in the execution environment)

## Examples

```
pip install multinmrfit
```

## Evaluation signals

- For each model and each parameter, the fitted value must match the simulated ground truth within the estimated standard deviation
- Recovery errors for intensity, area, chemical shift, linewidth, and coupling constant are computed per model and written to model_validation.csv with no missing entries
- All five multiplet models (singlet, doublet, triplet, doublet of doublets, quadruplet) reproduce the documented mixed Gaussian-Lorentzian equations
- Fitted linewidth (lw) values are physically valid (positive half-maximum linewidths) and Gaussian fraction a stays within its valid fractional range
- Estimated uncertainties (standard deviations) from MultiNMRFit are reported alongside every fitted parameter

## Limitations

- Validation is limited to the built-in mixed Gaussian-Lorentzian multiplet models singlet through quadruplet; user-defined models require separate handling
- The approach assumes the simulated ground truth exactly follows the documented equations, so it cannot detect systematic errors shared by the simulator and the fitter
- No discussion-section limitations, missing artifacts, or reproducibility gaps were extracted for this article

## Evidence

- [other] Mixed Gaussian-Lorentzian parameterization of multiplet models: [evidence span withheld — anchor status: not_found]
- [readme] Fitting purpose of the tool: "MultiNMRFit is a scientific tool designed to extract quantitative information (chemical shifts, signal intensity, coupling constants, etc) by fitting a serie of 1D spectra"
- [readme] Parameters estimated per signal: "estimation of several parameters for each signal of interest (**intensity**, **area**, **chemical shift**, **linewidth**, **coupling constant(s)**, etc),"
- [readme] Uncertainty estimation supporting the within-sigma validation criterion: "estimation of **uncertainty** on estimated parameters (standard deviation),"
