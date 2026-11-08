---
name: nmr-fit-uncertainty-estimation
description: Use when 1D or pseudo-2D NMR spectra contain overlapping multiplet peaks
  (e.g. a triplet plus a quadruplet) or low signal-to-noise, and you need quantitative
  parameters with reported uncertainty rather than simple integration.
when_to_use_negative:
- Signals are isolated singlets with no overlap or multiplet structure, where simple
  integration suffices
- Spectra are not 1D or pseudo-2D NMR data (e.g. imaging or non-NMR spectroscopy),
  which MultiNMRFit's multiplet signal models do not target
- You need fully automated, hands-off analysis without visual inspection of the fitted
  curves
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: MultiNMRFit
  role: Fits series of 1D/pseudo-2D NMR spectra with built-in or user-defined multiplet
    signal models and estimates parameter uncertainties
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
- name: pip
  role: Installs multinmrfit from GitHub/PyPI
- name: Python 3.8
  role: Runtime environment required by MultiNMRFit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:fb238e303ad1ef889a12b5ea3530ee002833c61d2190c692a8959eb08dda2371
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNMRFit
  tools:
  - MultiNMRFit
  - Python
  - Python 3.8
  - pip
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# nmr-fit-uncertainty-estimation

## Summary

Fit 1D or pseudo-2D NMR spectra with MultiNMRFit multiplet signal models to estimate intensity, area, chemical shift, linewidth and coupling constant(s), together with standard-deviation uncertainty, for overlapping multiplet signals such as triplets and quadruplets.

## When to use

Use when 1D or pseudo-2D NMR spectra contain overlapping multiplet peaks (e.g. a triplet plus a quadruplet) or low signal-to-noise, and you need quantitative parameters with reported uncertainty rather than simple integration.

## When NOT to use

- Signals are isolated singlets with no overlap or multiplet structure, where simple integration suffices
- Spectra are not 1D or pseudo-2D NMR data (e.g. imaging or non-NMR spectroscopy), which MultiNMRFit's multiplet signal models do not target
- You need fully automated, hands-off analysis without visual inspection of the fitted curves

## Inputs

- Series of 1D NMR spectra (individual 1D, pseudo-2D, or tabulated text files)
- Peak picking / multiplicity definitions for each signal of interest (e.g. triplet, quadruplet)

## Outputs

- Fitted parameters per signal: intensity, area, chemical shift, linewidth, coupling constant(s)
- Standard deviation (uncertainty) on each estimated parameter
- fitting_results.csv of parameter values with standard deviations
- Visual inspection plots of fitted curves

## How to apply

Install MultiNMRFit with `pip install multinmrfit` under Python 3.8+ and load the spectra as a series of 1D spectra (acquired individually, as pseudo-2D, or as tabulated text files). Use the semi-automated peak picking and multiplicity definition to assign multiplet signal models (built-in or user-defined) to each signal of interest, accounting for peak overlaps and zero-order baseline correction. Run the fit to estimate intensity, area, chemical shift, linewidth and coupling constant(s) per signal, then visually inspect the fitted curves and record the estimated standard deviation on each parameter. The task's synthetic triplet+quadruplet benchmark showed robust estimation even under low SNR or significant overlap, with fitting whole multiplets improving parameter accuracy and precision. Write fitted values with standard deviations to fitting_results.csv.

## Related tools

- **MultiNMRFit** (Fits series of 1D/pseudo-2D NMR spectra with built-in or user-defined multiplet signal models and estimates parameter uncertainties) — https://github.com/NMRTeamTBI/MultiNMRFit/
- **pip** (Installs multinmrfit from GitHub/PyPI)
- **Python 3.8** (Runtime environment required by MultiNMRFit)

## Examples

```
pip install multinmrfit; nmrfit
```

## Evaluation signals

- Standard deviations (uncertainties) are returned for every fitted parameter (intensity, area, chemical shift, linewidth, coupling constant(s))
- Visual inspection of fitted curves shows the triplet and quadruplet multiplets are reproduced, including overlapping peaks and zero-order baseline correction
- Parameter accuracy and precision improve relative to fitting peaks individually when multiplets are fitted as whole models
- All parameter values with standard deviations are present in fitting_results.csv with one row per signal per spectrum in the series

## Limitations

- Requires Python 3.8 or higher; tested on Unix, Windows and MacOS platforms
- Analysis is semi-automated: peak picking and multiplicity definition require user input and visual inspection of fitted curves
- The repository discussion section contains no changelog, so limitations, missing artifacts, or reproducibility gaps could not be extracted from it

## Evidence

- [abstract] MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models: "MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models"
- [readme] semi-automated analysis for peak picking and definition of multiplicity for each signal: "**semi-automated analysis** for **peak picking** and **definition of multiplicity** for each signal,"
- [readme] estimation of uncertainty on estimated parameters (standard deviation): "estimation of **uncertainty** on estimated parameters (standard deviation),"
- [methods] MultiNMRFit requires Python 3.8 or higher and run on all platforms (Windows, MacOS and Unix): "MultiNMRFit requires Python 3.8 or higher and run on all platforms (Windows, MacOS and Unix)"
- [readme] account for overlaps between peaks and zero-order baseline correction: "account for **overlaps** between peaks and **zero-order baseline correction**,"
