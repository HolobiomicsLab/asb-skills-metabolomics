---
name: mixed-gaussian-lorentzian-lineshape-fitting
description: Use when quantitative parameters (intensity, area, chemical shift, linewidth,
  J coupling constants) must be extracted from 1D or pseudo-2D NMR spectra of multiplets
  whose lineshape is neither purely Gaussian nor purely Lorentzian.
when_to_use_negative:
- The spectrum is 2D (not 1D or pseudo-2D) — MultiNMRFit is designed for series of
  1D spectra
- Only peak positions are needed without quantitative intensity/area/coupling estimation
- A lineshape model outside the built-in singlet–quadruplet multiplets is required
  and no user-defined model has been implemented
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3172
tools:
- name: MultiNMRFit
  role: Provides the built-in mixed Gaussian-Lorentzian multiplet signal models and
    performs the nonlinear fitting with uncertainty estimation
  repo: https://github.com/NMRTeamTBI/MultiNMRfit/
- name: Python 3.8
  role: Runtime environment required to install and run multinmrfit as a library or
    GUI
- name: pip
  role: Installation channel for the multinmrfit package (`pip install multinmrfit`)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:8cec74c62ba363ca09db24aef610b6bfda15126821827caa27c293c73a9d9441
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNMRfit
  tools:
  - MultiNMRFit
  - Python
  - Python 3.8
  - pip
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# mixed-gaussian-lorentzian-lineshape-fitting

## Summary

Fit 1D/pseudo-2D NMR multiplet signals (singlet through quadruplet) with MultiNMRFit's built-in mixed Gaussian-Lorentzian signal models to recover intensity, area, chemical shift, linewidth, and coupling constants with estimated standard deviations.

## When to use

Use when quantitative parameters (intensity, area, chemical shift, linewidth, J coupling constants) must be extracted from 1D or pseudo-2D NMR spectra of multiplets whose lineshape is neither purely Gaussian nor purely Lorentzian.

## When NOT to use

- The spectrum is 2D (not 1D or pseudo-2D) — MultiNMRFit is designed for series of 1D spectra
- Only peak positions are needed without quantitative intensity/area/coupling estimation
- A lineshape model outside the built-in singlet–quadruplet multiplets is required and no user-defined model has been implemented

## Inputs

- 1D NMR spectra acquired individually or as pseudo-2D experiments (all nuclei, e.g. 1H, 13C, 15N, 31P)
- Tabulated text files of spectra
- Initial peak-picking positions and multiplicity definitions per signal (semi-automated)

## Outputs

- Fitted signal parameters per multiplet: intensity, area, chemical shift, linewidth, coupling constant(s)
- Estimated uncertainties (standard deviations) on each fitted parameter
- Fitted curves for visual inspection

## How to apply

Install multinmrfit (requires Python 3.8+) via pip and either launch the graphical interface with `nmrfit` or use it as a Python library. For each signal of interest, select one of the built-in mixed Gaussian-Lorentzian multiplet models (singlet, doublet, triplet, doublet of doublets, quadruplet), which parameterize intensity via the chemical shift xs, total signal intensity I, scalar coupling constant J, linewidth at half maximum lw, and the Gaussian fraction a. Use the semi-automated peak picking and multiplicity definition to initialize signals, allowing for peak overlaps and zero-order baseline correction. Fit the experimental spectrum(s), then inspect fitted curves visually and check that each fitted parameter matches expectations within its estimated standard deviation. For validation, simulate synthetic spectra with known ground-truth xs, I, J, lw, and a values, refit them, and confirm recovery within the estimated standard deviation per parameter.

## Related tools

- **MultiNMRFit** (Provides the built-in mixed Gaussian-Lorentzian multiplet signal models and performs the nonlinear fitting with uncertainty estimation) — https://github.com/NMRTeamTBI/MultiNMRfit/
- **Python 3.8** (Runtime environment required to install and run multinmrfit as a library or GUI)
- **pip** (Installation channel for the multinmrfit package (`pip install multinmrfit`))

## Examples

```
pip install multinmrfit; nmrfit
```

## Evaluation signals

- For each model and parameter, the fitted value must match the simulated ground truth within the estimated standard deviation
- Recovery error for intensity, area, chemical shift, linewidth, and coupling constant is computed per multiplet model and tabulated (e.g. model_validation.csv)
- Visual inspection of fitted curves shows the fitted lineshape overlapping the experimental peaks, including overlapped peaks
- Standard deviations are reported for every estimated parameter, confirming uncertainty estimation ran
- Fitted scalar coupling constants and chemical shifts are consistent with the multiplicity assigned during peak picking

## Limitations

- Zero-order baseline correction only — more complex baselines are not handled
- Built-in models cover singlet through quadruplet; other multiplicities require user-defined signal models
- Overlap between peaks must be accounted for by the fitting model; heavily overlapped regions may be challenging
- Requires Python 3.8 or higher

## Evidence

- [results] Mixed Gaussian-Lorentzian parameterization: [evidence span withheld — anchor status: not_found]
- [abstract] Scope of fitting tool: "MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models"
- [readme] Feature list: "estimation of several parameters for each signal of interest (**intensity**, **area**, **chemical shift**, **linewidth**, **coupling constant(s)**, etc)"
- [readme] Overlap and baseline handling: "account for **overlaps** between peaks and **zero-order baseline correction**"
- [methods] Installation command: "Use `pip` to **install PhysioFit from GitHub**:

bash
$ pip install multinmrfit"
- [abstract] Demonstrated robustness: "We demonstrate the robustness and flexibility of MultiNMRFit on 1H, 13C, and 31P NMR datasets collected in metabolomics and isotope labeling studies."
