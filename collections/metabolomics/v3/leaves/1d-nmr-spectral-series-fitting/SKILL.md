---
name: 1d-nmr-spectral-series-fitting
description: Use when a series of 1D NMR spectra (individual, pseudo-2D, or tabulated
  text files) must be deconvoluted into overlapping multiplet components to quantify
  metabolites or isotopomer/isotopic-species concentrations over time or across samples.
when_to_use_negative:
- Data are 2D/3D NMR spectra requiring full multidimensional processing — MultiNMRFit
  fits series of 1D spectra (individual, pseudo-2D, or tabulated text) only
- The goal is routine univariate peak integration where multiplet models, overlap
  handling, and baseline correction are unnecessary
- Fitting requires signal models not among the built-ins and the user cannot define
  custom models (though user-defined models are supported)
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MultiNMRFit
  role: Fits series of 1D/pseudo-2D NMR spectra with built-in multiplet signal models
    to extract signal parameters and uncertainties; used for isotopic-species quantification
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
- name: pip
  role: Installs the multinmrfit package into the Python 3 environment
- name: Python 3
  role: Runtime environment (>=3.8) in which MultiNMRFit runs as a library with a
    graphical user interface
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:245558df8b5dc1cd6f57aaba3420db64878987b2aa95ddbaf5077af217e8e535
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNMRFit
  tools:
  - MultiNMRFit
  - Python
  - Python 3
  - pip
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# 1D NMR spectral series fitting

## Summary

Fit series of 1D or pseudo-2D NMR spectra (1H, 13C, 31P) with MultiNMRFit's multiplet signal models to extract quantitative parameters (intensity, area, chemical shift, coupling constants) with standard-deviation uncertainties, e.g. for isotopic-species quantification in isotope labeling studies.

## When to use

Use when a series of 1D NMR spectra (individual, pseudo-2D, or tabulated text files) must be deconvoluted into overlapping multiplet components to quantify metabolites or isotopomer/isotopic-species concentrations over time or across samples.

## When NOT to use

- Data are 2D/3D NMR spectra requiring full multidimensional processing — MultiNMRFit fits series of 1D spectra (individual, pseudo-2D, or tabulated text) only
- The goal is routine univariate peak integration where multiplet models, overlap handling, and baseline correction are unnecessary
- Fitting requires signal models not among the built-ins and the user cannot define custom models (though user-defined models are supported)

## Inputs

- Series of 1D NMR spectra acquired individually or as a pseudo-2D spectrum (e.g. 1H-13C HSQC labeling dataset)
- Tabulated text files of 1D spectra
- Semi-automated peak-picking definitions of multiplicity for each signal
- Calibration for converting signal areas to concentrations (optional)

## Outputs

- Estimated parameters per fitted signal: intensity, area, chemical shift, linewidth, coupling constant(s)
- Standard-deviation uncertainties on estimated parameters
- Time courses of metabolite/isotopic-species concentrations (e.g. acetate_quantification.csv, aspartate_hsqc_fit_results.csv)

## How to apply

Install MultiNMRFit in a Python 3 (>=3.8) environment with pip (`pip install multinmrfit`) and launch the GUI with `nmrfit`, or use it as a Python library. Load the spectral series (e.g. a pseudo-2D 1H-13C HSQC dataset or a pseudo-2D series covering the acetate methyl region), then use the semi-automated peak picking to place signals and define each signal's multiplicity. Fit each signal across the series with the built-in multiplet signal models, accounting for overlaps between peaks and applying zero-order baseline correction; the tool can use all nuclei (1H, 13C, 15N, 31P). Extract the estimated parameters (intensity, area, chemical shift, coupling constants) together with their standard-deviation uncertainties, and convert fitted areas to concentrations with a calibration where required (e.g. acetate isotopomer time courses in mM). Judge success by visual inspection of fitted curves against experimental spectra and, for validation datasets, by agreement between quantified isotopic species and the predicted labeling pattern (differences below 1%, total acetate stable at 8.5 ± 0.1 mM).

## Related tools

- **MultiNMRFit** (Fits series of 1D/pseudo-2D NMR spectra with built-in multiplet signal models to extract signal parameters and uncertainties; used for isotopic-species quantification) — https://github.com/NMRTeamTBI/MultiNMRFit/
- **pip** (Installs the multinmrfit package into the Python 3 environment)
- **Python 3** (Runtime environment (>=3.8) in which MultiNMRFit runs as a library with a graphical user interface)

## Examples

```
pip install multinmrfit && nmrfit
```

## Evaluation signals

- Quantified isotopic species agree with the predicted labeling pattern with differences below 1% (aspartate 1H-13C HSQC standard validation)
- Total metabolite concentration invariant across the series (e.g. total acetate stable at 8.5 ± 0.1 mM)
- Fitted curves pass visual inspection against the experimental spectra
- Estimated parameters are reported with standard-deviation uncertainties that are reasonable and finite
- Expected kinetic behavior reproduced, e.g. 2-13C1-acetate declining from 8.5 to 6.5 mM while glucose-derived labeled acetate rises from 0.1 to 1.9 mM

## Limitations

- Requires Python 3.8 or higher; tested on Unix, Windows, and MacOS platforms
- The extraction article section contains no discussion text: no limitations, failure modes, or reproducibility gaps are explicitly documented
- Fitting is limited to series of 1D spectra; it is not a full multidimensional NMR processing tool

## Evidence

- [abstract] MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models that account for: "MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models"
- [readme] semi-automated analysis: "**semi-automated analysis** for **peak picking** and **definition of multiplicity** for each signal,"
- [readme] account for overlaps: "account for **overlaps** between peaks and **zero-order baseline correction**,"
- [methods] MultiNMRFit requires Python 3.8 or higher and run on all platforms (Windows, MacOS and Unix): "MultiNMRFit requires Python 3.8 or higher and run on all platforms (Windows, MacOS and Unix)"
- [abstract] robustness on isotope labeling datasets: "We demonstrate the robustness and flexibility of MultiNMRFit on 1H, 13C, and 31P NMR datasets collected in metabolomics and isotope labeling studies."
