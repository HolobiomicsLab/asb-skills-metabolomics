---
name: isotopic-species-quantification-from-hsqc
description: Use when a pseudo-2D (e.g. 1H-13C HSQC) isotope-labeling NMR series must
  be decomposed into individual isotopic species and their relative abundances quantified,
  especially with overlapping peaks.
when_to_use_negative:
- Input is a set of individual 1D spectra unrelated to isotope-labeling quantification
  of isotopic species from an HSQC series
- The sample lacks a known or predictable labeling pattern against which quantification
  can be validated
- A fully automated (non-semi-automated) peak-picking analysis with no user-defined
  signal multiplicity is required
tools:
- name: MultiNMRFit
  role: Fits each multiplet signal of the 1H-13C HSQC series with built-in multiplet
    models, handles peak overlaps and zero-order baseline correction, and outputs
    intensity/area/chemical-shift estimates with uncertainties
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
- name: pip
  role: Used to install multinmrfit into the Python 3 environment
- name: Python
  role: Runtime environment (Python 3.8 or higher) in which MultiNMRFit is installed
    and can be used as a library
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:2452113f90cabd186581ab367f8e0aeca036c4dbde40a11c11f6d92ced277aa6
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
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# isotopic-species-quantification-from-hsqc

## Summary

Quantify isotopic species of metabolites (e.g. aspartate) from 1H-13C HSQC pseudo-2D spectra by fitting every peak of each multiplet with MultiNMRFit, a Python 3 open-source tool that estimates intensity, area, chemical shift, and coupling constants with standard-deviation uncertainties. Validated by agreement with predicted 13C-labeling patterns within 1%.

## When to use

Use when a pseudo-2D (e.g. 1H-13C HSQC) isotope-labeling NMR series must be decomposed into individual isotopic species and their relative abundances quantified, especially with overlapping peaks.

## When NOT to use

- Input is a set of individual 1D spectra unrelated to isotope-labeling quantification of isotopic species from an HSQC series
- The sample lacks a known or predictable labeling pattern against which quantification can be validated
- A fully automated (non-semi-automated) peak-picking analysis with no user-defined signal multiplicity is required

## Inputs

- pseudo-2D 1H-13C HSQC isotope-labeling NMR spectral series (aspartate standard dataset with known, predictable labeling pattern)
- predicted 13C-labeling pattern of the standard sample for validation

## Outputs

- fitted parameters per isotopic species: intensity, area, chemical shift, with standard-deviation uncertainties (e.g. aspartate_hsqc_fit_results.csv)
- visually inspectable fitted curves overlaid on the experimental HSQC series

## How to apply

Install MultiNMRFit into a Python 3.8+ environment with pip, then load the aspartate 1H-13C HSQC series (pseudo-2D dataset) and use its semi-automated peak picking to define the multiplicity of each signal. Fit each signal across the spectral series with the built-in multiplet signal models, accounting for overlaps between peaks and applying zero-order baseline correction. Extract the estimated parameters (intensity, area, chemical shift) for each isotopic species together with their standard-deviation uncertainties (e.g. to aspartate_hsqc_fit_results.csv). Validation requires the quantified isotopic species to agree with the predicted labeling pattern with differences below 1% and fitted curves to pass visual inspection.

## Related tools

- **MultiNMRFit** (Fits each multiplet signal of the 1H-13C HSQC series with built-in multiplet models, handles peak overlaps and zero-order baseline correction, and outputs intensity/area/chemical-shift estimates with uncertainties) — https://github.com/NMRTeamTBI/MultiNMRFit/
- **pip** (Used to install multinmrfit into the Python 3 environment)
- **Python** (Runtime environment (Python 3.8 or higher) in which MultiNMRFit is installed and can be used as a library)

## Examples

```
$ pip install multinmrfit
$ nmrfit
```

## Evaluation signals

- Quantified isotopic species abundances agree with the predicted labeling pattern of the standard sample with differences below 1%
- Each fitted curve passes visual inspection against the experimental HSQC series
- Estimated parameters (intensity, area, chemical shift) are reported together with standard-deviation uncertainties
- Signal multiplicities are consistently defined via semi-automated peak picking for every peak in the multiplet
- Output table (aspartate_hsqc_fit_results.csv) contains all fitted parameters for each isotopic species across the series

## Limitations

- Analysis is semi-automated: peak picking and definition of multiplicity for each signal require user involvement
- No discussion section text was available, so no additional published limitations could be extracted
- Requires Python 3.8 or higher; runs on Windows, MacOS, and Unix

## Evidence

- [abstract] abstract: "We present MultiNMRFit, an open-source software designed for high-throughput analysis of 1D NMR spectra, whether acquired individually or as pseudo-2D experiments."
- [methods] methods: "MultiNMRFit is a scientific tool designed to extract quantitative information (chemical shifts, signal intensity, coupling constants, etc) by fitting a serie of 1D spectra"
- [readme] readme: "account for **overlaps** between peaks and **zero-order baseline correction**"
