---
name: j-coupling-multiplet-structure-analysis
description: Use when 1D or pseudo-2D NMR spectra (1H, 13C, 15N, 31P, etc.) contain
  multiplet signals whose chemical shift, intensity, area, linewidth, and J coupling
  constants must be quantified, including overlapping peaks.
when_to_use_negative:
- The spectra are full multi-dimensional NMR correlation experiments rather than series
  of 1D or pseudo-2D spectra that MultiNMRFit fits
- The signal shape cannot be captured by the built-in multiplet models and no user-defined
  model has been implemented
- Only qualitative peak identification is needed — the skill targets quantitative
  parameter extraction (intensity, area, chemical shift, J), not annotation alone
edam_operation: null
tools:
- name: MultiNMRFit
  role: Provides the built-in mixed Gaussian-Lorentzian multiplet signal models (singlet
    to quadruplet), semi-automated peak picking, overlap/baseline handling, and uncertainty
    estimation used to fit the spectra
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
- name: Python 3.8
  role: Runtime environment for installing multinmrfit via pip and using MultiNMRFit
    as a library to run the multiplet fits
- name: pip
  role: Installs the multinmrfit package (pip install multinmrfit) in the execution
    environment
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:9df83a7f0a4b5f918f8b6accbd3bd99745340ead44c9351d58f26a6b71f193f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNMRFit
  tools:
  - MultiNMRFit
  - Python
  - Python 3.8
  - pip
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# j-coupling-multiplet-structure-analysis

## Summary

Fit built-in mixed Gaussian-Lorentzian multiplet signal models (singlet through quadruplet) to 1D or pseudo-2D NMR spectra with MultiNMRFit to recover chemical shift (xs), total intensity (I), scalar coupling constant (J), linewidth at half maximum (lw), and the Gaussian fraction (a) per signal.

## When to use

Use when 1D or pseudo-2D NMR spectra (1H, 13C, 15N, 31P, etc.) contain multiplet signals whose chemical shift, intensity, area, linewidth, and J coupling constants must be quantified, including overlapping peaks.

## When NOT to use

- The spectra are full multi-dimensional NMR correlation experiments rather than series of 1D or pseudo-2D spectra that MultiNMRFit fits
- The signal shape cannot be captured by the built-in multiplet models and no user-defined model has been implemented
- Only qualitative peak identification is needed — the skill targets quantitative parameter extraction (intensity, area, chemical shift, J), not annotation alone

## Inputs

- 1D NMR spectra acquired individually, as a pseudo-2D spectrum, or provided as tabulated text files
- Peak-picking / multiplicity definitions for each signal of interest
- Choice of multiplet signal model (singlet, doublet, triplet, doublet of doublets, quadruplet) or a user-defined model
- Optional ground-truth parameter values (xs, I, J, lw, a) for simulated validation spectra

## Outputs

- Fitted signal parameters: intensity, area, chemical shift, linewidth, coupling constant(s), with estimated standard deviations
- Fitted curves for visual inspection
- Parameter recovery error table (e.g. model_validation.csv) when validating against simulated ground truth

## How to apply

Install multinmrfit (Python 3.8+, pip) and either drive the graphical interface via `nmrfit` or use it as a Python library. For each signal, use the semi-automated peak picking and multiplicity definition, then select one of the five built-in mixed Gaussian-Lorentzian models (singlet, doublet, triplet, doublet of doublets, quadruplet) parameterized by xs (chemical shift), I (total signal intensity), J (scalar coupling constant), lw (linewidth at half maximum), and a (fraction of Gaussian part of the signal). Fit the spectrum, letting the model account for overlaps between peaks and zero-order baseline correction, and inspect the fitted curves visually. Record the fitted parameters together with their estimated standard deviations; for a ground-truth validation, simulate synthetic spectra with predefined xs, I, J, lw, and a values, refit, and compute recovery error per parameter per model (e.g. into model_validation.csv).

## Related tools

- **MultiNMRFit** (Provides the built-in mixed Gaussian-Lorentzian multiplet signal models (singlet to quadruplet), semi-automated peak picking, overlap/baseline handling, and uncertainty estimation used to fit the spectra) — https://github.com/NMRTeamTBI/MultiNMRFit/
- **Python 3.8** (Runtime environment for installing multinmrfit via pip and using MultiNMRFit as a library to run the multiplet fits)
- **pip** (Installs the multinmrfit package (pip install multinmrfit) in the execution environment)

## Examples

```
pip install multinmrfit && nmrfit
```

## Evaluation signals

- For each multiplet model and each parameter (intensity, area, chemical shift, linewidth, coupling constant), the fitted value must match the simulated ground truth within the estimated standard deviation
- Parameter recovery errors per model are tabulated (e.g. written to model_validation.csv) and compared across multiplicities
- Visual inspection of fitted curves shows the modeled multiplet traces the experimental peaks, including overlap regions
- Each fitted parameter is reported with an estimated standard deviation (uncertainty estimate)
- Fitted J values are consistent across the series of 1D spectra in a pseudo-2D experiment

## Limitations

- Baseline handling is limited to zero-order baseline correction, so strongly curved baselines may degrade fits
- Uncertainty is reported only as standard deviation on estimated parameters; no other confidence measures are described
- The built-in models cover singlet through quadruplet multiplet structure; higher-order or non-standard lineshapes require user-defined models
- No discussion-section limitations or reproducibility gaps could be extracted from the available article text

## Evidence

- [other] Mixed GL multiplet parameterization: [evidence span withheld — anchor status: not_found]
- [abstract] Extraction by fitting signal models: "MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models"
- [readme] Tool purpose and fitting scope: "MultiNMRFit is a scientific tool designed to extract quantitative information (chemical shifts, signal intensity, coupling constants, etc) by fitting a serie of 1D spectra"
- [readme] Overlap and baseline handling: "account for **overlaps** between peaks and **zero-order baseline correction**"
- [readme] Uncertainty estimation: "estimation of **uncertainty** on estimated parameters (standard deviation)"
- [readme] Semi-automated multiplicity definition: "**semi-automated analysis** for **peak picking** and **definition of multiplicity** for each signal"
