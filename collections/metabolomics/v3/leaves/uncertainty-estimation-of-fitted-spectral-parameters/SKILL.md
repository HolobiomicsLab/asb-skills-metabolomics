---
name: uncertainty-estimation-of-fitted-spectral-parameters
description: Use when a series of 1D or pseudo-2D NMR spectra (any nucleus, e.g. 1H,
  13C, 31P) needs per-signal fitted parameters with reported standard-deviation uncertainties,
  such as monitoring metabolite signals over a real-time assay.
when_to_use_negative:
- The data are full multi-dimensional (nD) NMR spectra rather than a series of 1D
  or pseudo-2D spectra, which is outside MultiNMRFit's scope
- Only raw peak positions are needed without quantitative fitting or uncertainty estimates
  (use simple peak picking instead)
- The spectra require higher-order baseline correction beyond zero-order baseline,
  which is what the models account for
edam_operation: http://edamontology.org/operation_3799
tools:
- name: MultiNMRFit
  role: Performs the semi-automated peak picking, multiplet signal fitting, and estimation
    of parameters with standard-deviation uncertainties for 1D and pseudo-2D NMR spectra
  repo: https://github.com/NMRTeamTBI/MultiNMRfit/
- name: pip
  role: Used to install the multinmrfit package
- name: Python
  role: Runtime environment (Python 3.8+); MultiNMRFit is also usable as a Python
    library
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:5e3d288d6a0b3d1db0aacd8f0824d2c0621d258e894cc1c90ccfae8c62b465e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNMRfit
  tools:
  - MultiNMRFit
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# uncertainty-estimation-of-fitted-spectral-parameters

## Summary

Estimates NMR signal parameters (intensity, area, chemical shift, linewidth, coupling constants) together with their standard-deviation uncertainties by fitting 1D or pseudo-2D spectra with MultiNMRFit's built-in multiplet signal models, demonstrated on 1H, 13C, and 31P datasets such as real-time glycolysis assays.

## When to use

Use when a series of 1D or pseudo-2D NMR spectra (any nucleus, e.g. 1H, 13C, 31P) needs per-signal fitted parameters with reported standard-deviation uncertainties, such as monitoring metabolite signals over a real-time assay.

## When NOT to use

- The data are full multi-dimensional (nD) NMR spectra rather than a series of 1D or pseudo-2D spectra, which is outside MultiNMRFit's scope
- Only raw peak positions are needed without quantitative fitting or uncertainty estimates (use simple peak picking instead)
- The spectra require higher-order baseline correction beyond zero-order baseline, which is what the models account for

## Inputs

- Series of 1D NMR spectra acquired individually or as a pseudo-2D experiment (e.g. 31P spectra from a real-time glycolysis assay)
- Tabulated text files of spectra
- User-defined signal models (optional, added to built-in multiplet models)

## Outputs

- Parameter table per signal (intensity, area, chemical shift, linewidth, coupling constants) with standard-deviation uncertainties, e.g. 31p_fit_results.csv
- Fitted-spectra plot with fitted curves overlaid on experimental spectra, e.g. 31p_fitted_spectra.png

## How to apply

Install MultiNMRFit via pip (Python 3.8+) and launch the nmrfit graphical interface or use it as a Python library. Load the spectra series (acquired individually, as a pseudo-2D experiment, or as tabulated text files), then use the semi-automated peak picking to detect resonance positions and define multiplicity for each signal of interest. Fit each spectrum with the built-in multiplet signal models, which account for overlaps between peaks and zero-order baseline correction. The fit returns estimated intensity, area, chemical shift, linewidth, and coupling constant(s) for each signal, each with an estimated standard deviation as its uncertainty. Inspect the fitted curves overlaid on the experimental spectra visually — the article prescribes this as the acceptance criterion — and export the parameter table (e.g. 31p_fit_results.csv) and fitted-spectra plot (e.g. 31p_fitted_spectra.png).

## Related tools

- **MultiNMRFit** (Performs the semi-automated peak picking, multiplet signal fitting, and estimation of parameters with standard-deviation uncertainties for 1D and pseudo-2D NMR spectra) — https://github.com/NMRTeamTBI/MultiNMRfit/
- **pip** (Used to install the multinmrfit package)
- **Python** (Runtime environment (Python 3.8+); MultiNMRFit is also usable as a Python library)

## Examples

```
pip install multinmrfit; nmrfit
```

## Evaluation signals

- Visual inspection confirms fitted curves overlay the experimental spectra for every spectrum in the series
- The optimization converged and each estimated parameter (intensity, area, chemical shift, linewidth, coupling constant) is reported with a standard-deviation uncertainty
- The exported parameter table (e.g. 31p_fit_results.csv) contains entries for all picked signals with their defined multiplicity
- Fitted parameter values (e.g. chemical shifts, coupling constants) are physically plausible for the nucleus and metabolites under study

## Limitations

- Acceptance is judged by visual inspection of fitted curves against experimental spectra rather than a quantitative threshold
- Baseline handling is limited to zero-order baseline correction
- No discussion-section limitations text was available in the article context (no changelog found)

## Evidence

- [abstract] MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants): "MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models"
- [abstract] We demonstrate the robustness and flexibility of MultiNMRFit on 1H, 13C, and 31P NMR datasets: "We demonstrate the robustness and flexibility of MultiNMRFit on 1H, 13C, and 31P NMR datasets collected in metabolomics and isotope labeling studies."
- [readme] estimation of uncertainty on estimated parameters (standard deviation): "estimation of **uncertainty** on estimated parameters (standard deviation),"
- [readme] account for overlaps between peaks and zero-order baseline correction: "account for **overlaps** between peaks and **zero-order baseline correction**,"
