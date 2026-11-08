---
name: isotope-labeling-real-time-kinetics-analysis
description: Use when monitoring reactant-to-product conversion of isotope-labeled
  metabolites (1H, 13C, 31P) across a series of 1D or pseudo-2D NMR spectra and extracting
  time-resolved concentrations.
when_to_use_negative:
- The data are not a series of 1D or pseudo-2D NMR spectra (e.g., full 2D NMR spectra
  requiring multidimensional peak deconvolution)
- The goal is quantitative NMR of a single, static spectrum rather than time-course
  kinetics of labeled metabolite conversion
- Signals require multiplet models beyond the built-in ones and no user-defined model
  has been implemented for the observed spin system
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3292
tools:
- name: MultiNMRFit
  role: Semi-automated peak picking, multiplet model fitting of each spectrum in the
    series, overlap handling, baseline correction, and extraction of time-resolved
    signal parameters with uncertainties
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
- name: Python
  role: Runtime environment for MultiNMRFit (Python 3.8 or higher); MultiNMRFit is
    also available as a Python library for scripted analysis
- name: pip
  role: Installation of the multinmrfit package
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:010fc05beece1a7f239c7a16c3a6df9c8b766c98609ee14b816441ff913509ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNMRFit
  tools:
  - MultiNMRFit
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3292
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# isotope-labeling-real-time-kinetics-analysis

## Summary

Quantify real-time conversion kinetics of isotope-labeled metabolites (e.g., uniformly 13C-labeled glucose to glucose-6-phosphate) by fitting multiplet signal models to series of 1D or pseudo-2D NMR spectra with MultiNMRFit.

## When to use

Use when monitoring reactant-to-product conversion of isotope-labeled metabolites (1H, 13C, 31P) across a series of 1D or pseudo-2D NMR spectra and extracting time-resolved concentrations.

## When NOT to use

- The data are not a series of 1D or pseudo-2D NMR spectra (e.g., full 2D NMR spectra requiring multidimensional peak deconvolution)
- The goal is quantitative NMR of a single, static spectrum rather than time-course kinetics of labeled metabolite conversion
- Signals require multiplet models beyond the built-in ones and no user-defined model has been implemented for the observed spin system

## Inputs

- Series of individually acquired 1D NMR spectra stacked as a pseudo-2D experiment (or tabulated text files)
- 13C-detected pseudo-2D spectra of uniformly 13C-labeled metabolites (e.g., GLC and G6P appearing as doublets of triplets)
- 1H, 13C, 15N, or 31P NMR spectra from metabolomics or isotope labeling studies

## Outputs

- Fitted intensity, area, chemical shift, linewidth, and coupling constant(s) for each signal at every time point
- Standard deviation (uncertainty) on each estimated parameter
- Real-time relative concentration dynamics plots of reactant and product
- Visually inspectable fitted curves overlaid on the spectra

## How to apply

Install MultiNMRFit (pip install multinmrfit, requires Python 3.8+) and load the pseudo-2D dataset, i.e., a series of individually acquired 1D spectra stacked as a pseudo-2D experiment. Run the semi-automated analysis for peak picking and definition of multiplicity, assigning appropriate multiplet models (e.g., doublet-of-triplets for uniformly 13C-labeled glucose and G6P). Fit each spectrum in the series with the built-in multiplet signal models, enabling overlap handling between peaks and zero-order baseline correction, then extract fitted intensity, area, chemical shift, and coupling constants with estimated standard deviations at every time point. Convert fitted areas to relative concentrations and plot the real-time dynamics of labeled reactant and product. Only accept the fit after visual inspection confirms each multiplet is correctly reproduced at every time point and the reported parameter standard deviation is reasonable.

## Related tools

- **MultiNMRFit** (Semi-automated peak picking, multiplet model fitting of each spectrum in the series, overlap handling, baseline correction, and extraction of time-resolved signal parameters with uncertainties) — https://github.com/NMRTeamTBI/MultiNMRFit/
- **Python** (Runtime environment for MultiNMRFit (Python 3.8 or higher); MultiNMRFit is also available as a Python library for scripted analysis)
- **pip** (Installation of the multinmrfit package)

## Examples

```
pip install multinmrfit; nmrfit
```

## Evaluation signals

- Visual inspection of the fitted curves confirms each multiplet (e.g., doublet-of-triplets) is correctly reproduced at every time point in the series
- The parameter uncertainty (standard deviation) reported by the tool for each fitted signal is reasonable
- Fitted areas convert to relative concentrations whose real-time dynamics are consistent with substrate consumption and product accumulation
- Fit converges with overlap handling and zero-order baseline correction enabled across all spectra in the pseudo-2D series

## Limitations

- Requires Python 3.8 or higher
- The signal models must adequately represent the observed multiplicity; user-defined models may need to be added for non-standard spin systems
- No discussion section limitations, missing artifacts, or reproducibility gaps were extractable from the article

## Evidence

- [abstract] The 13C-detected pseudo-2D spectra of 13C-labeled GLC and G6P (both appearing as doublets of triplets) were fitted accurately with MultiNMRFit: [evidence span withheld — anchor status: not_found]
- [methods] MultiNMRFit is a scientific tool designed to extract quantitative information (chemical shifts, signal intensity, coupling constants, etc) by fitting a serie of 1D spectra: "MultiNMRFit is a scientific tool designed to extract quantitative information (chemical shifts, signal intensity, coupling constants, etc) by fitting a serie of 1D spectra"
- [readme] MultiNMRFit requires Python 3.8 or higher and run on all platforms (Windows, MacOS and Unix): "MultiNMRFit requires Python 3.8 or higher and run on all platforms (Windows, MacOS and Unix)."
- [readme] estimation of uncertainty on estimated parameters (standard deviation): "estimation of **uncertainty** on estimated parameters (standard deviation),"
