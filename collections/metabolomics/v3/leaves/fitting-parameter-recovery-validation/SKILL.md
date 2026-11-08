---
name: fitting-parameter-recovery-validation
description: Use when validating that a spectral fitting implementation (e.g. MultiNMRFit
  multiplet models) reproduces documented lineshape equations, by checking parameter
  recovery against simulated ground truth.
when_to_use_negative:
- The goal is fitting real experimental 1H, 13C, or 31P NMR spectra rather than verifying
  the fitting implementation against synthetic data
- The signal model under test is a user-defined model with no documented closed-form
  equation to reproduce
- Only pseudo-2D series fitting (with baseline and overlap handling) is needed and
  the lineshape equations themselves are already trusted
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MultiNMRFit
  role: Fitting engine whose built-in multiplet signal models are applied to the simulated
    spectra and whose estimated parameter standard deviations define the validation
    tolerance
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
- name: Python
  role: Language used to implement the mixed Gaussian-Lorentzian multiplet models
    and to simulate synthetic 1D spectra with ground-truth parameters
- name: pip
  role: Used to install the multinmrfit package in the Python 3.8+ execution environment
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:b0528fa81ee2fe6ae5e1a76717b76e1c5402ed3d300442ccef0585608a13e7ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNMRFit
  tools:
  - MultiNMRFit
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# fitting-parameter-recovery-validation

## Summary

A validation workflow for NMR spectral fitting software: implement the documented multiplet signal models (singlet through quadruplet) as mixed Gaussian-Lorentzian lineshapes, simulate synthetic 1D spectra with known ground-truth parameters, refit them with the tool under test, and confirm each fitted parameter matches ground truth within its estimated standard deviation.

## When to use

Use when validating that a spectral fitting implementation (e.g. MultiNMRFit multiplet models) reproduces documented lineshape equations, by checking parameter recovery against simulated ground truth.

## When NOT to use

- The goal is fitting real experimental 1H, 13C, or 31P NMR spectra rather than verifying the fitting implementation against synthetic data
- The signal model under test is a user-defined model with no documented closed-form equation to reproduce
- Only pseudo-2D series fitting (with baseline and overlap handling) is needed and the lineshape equations themselves are already trusted

## Inputs

- Python 3.8+ environment with multinmrfit installed via pip
- Implementations of the five multiplet signal models (singlet, doublet, triplet, doublet of doublets, quadruplet) parameterized by xs, I, J, lw, a
- Predefined ground-truth parameter values (xs, I, J, lw, a) for each multiplicity
- Simulated synthetic 1D NMR spectra generated from those ground-truth models

## Outputs

- model_validation.csv containing fitted vs. ground-truth parameters and per-parameter recovery errors
- Per-parameter estimated standard deviations from MultiNMRFit fits
- Pass/fail validation verdict per model and parameter (fitted value within one standard deviation of ground truth)

## How to apply

Implement the five multiplet signal models (singlet, doublet, triplet, doublet of doublets, quadruplet) as mixed Gaussian-Lorentzian lineshape functions parameterized by chemical shift xs, total intensity I, scalar coupling constant J, linewidth at half maximum lw, and Gaussian fraction a, exactly as documented for MultiNMRFit. Install multinmrfit in a Python 3.8+ environment via pip, then simulate synthetic 1D spectra using the implemented models with predefined ground-truth xs, I, J, lw, and a values for each multiplicity. Fit each simulated spectrum with MultiNMRFit's signal models and record fitted parameters alongside ground truth. For each model and parameter (intensity, area, chemical shift, linewidth, coupling constant), compute the recovery error and write the comparison table to model_validation.csv; validation succeeds when the fitted value matches the simulated ground truth within the estimated standard deviation, confirming the implementation reproduces the documented equations.

## Related tools

- **MultiNMRFit** (Fitting engine whose built-in multiplet signal models are applied to the simulated spectra and whose estimated parameter standard deviations define the validation tolerance) — https://github.com/NMRTeamTBI/MultiNMRFit/
- **Python** (Language used to implement the mixed Gaussian-Lorentzian multiplet models and to simulate synthetic 1D spectra with ground-truth parameters)
- **pip** (Used to install the multinmrfit package in the Python 3.8+ execution environment)

## Examples

```
pip install multinmrfit
```

## Evaluation signals

- For every multiplet model (singlet through quadruplet) and every parameter (intensity, area, chemical shift, linewidth, coupling constant), the fitted value matches the simulated ground truth within its estimated standard deviation
- model_validation.csv exists and contains recovery errors for all five models and all parameter types
- Fitted Gaussian-Lorentzian mixing coefficient a recovers the predefined ground-truth fraction of the Gaussian part of the signal
- Fitted coupling constants J match the ground-truth scalar coupling values used in simulation
- Fitted chemical shifts xs, intensities I, and linewidths lw agree with simulation ground truth to within reported standard deviations

## Limitations

- The README notes installation instructions require Python 3.8 or higher, so the validation environment must satisfy that requirement
- Validation relies on synthetic spectra; recovery on real experimental spectra with overlaps, baseline offsets, and noise is not established by this workflow
- The article's extracted discussion section contains no text, so no documented limitations or reproducibility gaps could be drawn from it

## Evidence

- [other] The mixed Gaussian-Lorentzian signal model describes intensity using parameters xs (chemical shift of the signal), I (total signal intensity), J (scalar coupling constant), lw (linewidth at half maximum), and a (fraction of Gaussian part of the signal).: [evidence span withheld — anchor status: not_found]
- [other] Validation criterion: fitted value matches simulated ground truth within estimated standard deviation, confirming the implementation reproduces documented equations.: [evidence span withheld — anchor status: not_found]
- [methods] MultiNMRFit extracts quantitative NMR parameters by fitting 1D spectra.: "MultiNMRFit is a scientific tool designed to extract quantitative information (chemical shifts, signal intensity, coupling constants, etc) by fitting a serie of 1D spectra"
- [readme] MultiNMRFit requires Python 3.8 or higher.: "MultiNMRFit requires Python 3.8 or higher and run on all platforms (Windows, MacOS and Unix)."
- [readme] MultiNMRFit estimates uncertainty (standard deviation) on estimated parameters.: "estimation of **uncertainty** on estimated parameters (standard deviation)"
