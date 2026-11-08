---
name: fitted-curve-visual-inspection
description: Use when MultiNMRFit has fitted 1D NMR spectra containing overlapping
  multiplet signals and fitted parameters (intensity, area, chemical shift, linewidth,
  coupling constants) must be quality-checked before being recorded.
when_to_use_negative:
- The spectra have not yet been fitted with multiplet signal models — this skill validates
  fits, it does not perform peak picking or model setup
- The data are 2D NMR spectra rather than 1D or pseudo-2D series of spectra, which
  is outside MultiNMRFit's stated scope
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: MultiNMRFit
  role: Provides the graphical user interface for visual inspection of fitted curves
    and estimation of parameter uncertainties after fitting 1D/pseudo-2D NMR spectra
  repo: https://github.com/NMRTeamTBI/MultiNMRfit/
- name: Python 3.8
  role: Runtime environment in which MultiNMRFit (shipped as a library and GUI) is
    installed and run
- name: pip
  role: Used to install multinmrfit from GitHub
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:524bdce8bb9d8f7564c864303a624128b22c9fc753afc629ee3e224996bd958c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNMRfit
  tools:
  - MultiNMRFit
  - Python
  - Python 3.8
  - pip
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# fitted-curve-visual-inspection

## Summary

Visually inspect the fitted curves produced by MultiNMRFit after fitting series of 1D (or pseudo-2D) NMR spectra, to confirm that multiplet signal models (e.g. triplets and quadruplets) correctly account for peak overlaps and baseline before trusting the estimated parameters and their standard deviations.

## When to use

Use when MultiNMRFit has fitted 1D NMR spectra containing overlapping multiplet signals and fitted parameters (intensity, area, chemical shift, linewidth, coupling constants) must be quality-checked before being recorded.

## When NOT to use

- The spectra have not yet been fitted with multiplet signal models — this skill validates fits, it does not perform peak picking or model setup
- The data are 2D NMR spectra rather than 1D or pseudo-2D series of spectra, which is outside MultiNMRFit's stated scope

## Inputs

- Fitted MultiNMRFit session on a series of 1D NMR spectra (individual 1D, pseudo-2D, or tabulated text files)
- Fitted multiplet signal models (triplet, quadruplet, or user-defined) with estimated parameters (intensity, area, chemical shift, linewidth, coupling constant(s)) and their standard deviations

## Outputs

- Visual overlay of fitted curves on experimental 1D NMR spectra
- Validated fitted parameter values with standard deviations, written e.g. to fitting_results.csv

## How to apply

After running the MultiNMRFit fit on a series of 1D spectra (individual 1D, pseudo-2D, or tabulated text files), use the semi-automated peak picking and multiplicity definition results to overlay the fitted multiplet models (triplet, quadruplet, or custom) on the experimental curves. Inspect that the fitted curves reproduce overlapping peaks and that zero-order baseline correction is adequate, since the software explicitly supports overlap accounting and baseline correction as model features. Pay particular attention to fits under challenging conditions — low signal-to-noise ratio or significant signal overlap — where parameter estimates and their standard deviations may be less reliable. Only accept fits whose visual residuals show no systematic mismatch, then record the estimated parameters with their uncertainties (e.g. to fitting_results.csv).

## Related tools

- **MultiNMRFit** (Provides the graphical user interface for visual inspection of fitted curves and estimation of parameter uncertainties after fitting 1D/pseudo-2D NMR spectra) — https://github.com/NMRTeamTBI/MultiNMRfit/
- **Python 3.8** (Runtime environment in which MultiNMRFit (shipped as a library and GUI) is installed and run)
- **pip** (Used to install multinmrfit from GitHub)

## Examples

```
pip install multinmrfit; nmrfit
```

## Evaluation signals

- Fitted curves visually match experimental multiplet peaks, including regions of significant overlap, with no systematic residual structure
- Zero-order baseline correction renders the residual baseline flat around each fitted signal
- Each estimated parameter is accompanied by a reported standard deviation (uncertainty) from the fit
- Fitted values (chemical shift, linewidth, coupling constants) are consistent across replicate spectra in the pseudo-2D series
- All validated parameter values with standard deviations are present in fitting_results.csv

## Limitations

- Visual inspection is qualitative; robustness under low signal-to-noise ratio or significant signal overlap may still yield elevated parameter uncertainties
- Fitting quality depends on correct semi-automated peak picking and multiplicity definition; misassigned multiplicity will produce visually poor fits
- No discussion/limitations section text was available in the source article, so further caveats could not be extracted

## Evidence

- [readme] Visual inspection of the fitted curves is a key feature listed for MultiNMRFit: "* **visual inspection of the fitted curves**,"
- [readme] Uncertainty estimation accompanies fitted parameters: "* estimation of **uncertainty** on estimated parameters (standard deviation),"
- [readme] The software accounts for peak overlaps and baseline correction, which the visual check must confirm: "* account for **overlaps** between peaks and **zero-order baseline correction**,"
- [abstract] MultiNMRFit extracts signal parameters by fitting spectra with built-in or user-defined signal models: "MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models"
- [methods] MultiNMRFit is a tool to extract quantitative information by fitting series of 1D spectra: "MultiNMRFit is a scientific tool designed to extract quantitative information (chemical shifts, signal intensity, coupling constants, etc) by fitting a serie of 1D spectra"
