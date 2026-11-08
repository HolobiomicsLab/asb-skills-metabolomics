---
name: overlapping-peak-deconvolution
description: Use when 1D or pseudo-2D NMR spectra contain overlapping multiplet signals
  (e.g. a triplet and quadruplet, or overlapping ADP/ATP peaks) and accurate peak
  parameters must be recovered by model-based fitting rather than integration.
when_to_use_negative:
- The spectra contain only well-resolved, non-overlapping peaks where simple integration
  suffices and multiplet modeling adds no accuracy
- The data are not NMR spectra (e.g. LC-MS peaks or chromatograms), since MultiNMRFit's
  multiplet signal models are designed for 1D NMR fitting
- Higher-order (non first-order) multiplet patterns require user-defined models not
  yet available in the software
tools:
- name: MultiNMRFit
  role: Open-source Python software used to pick peaks semi-automatically, define
    multiplicities, account for peak overlaps and zero-order baseline, and fit 1D/pseudo-2D
    NMR spectra with multiplet signal models
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
- name: pip
  role: Package installer used to install multinmrfit from GitHub/PyPI
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:f2c7110df56bd24f5bb04f4e1dcae26c9283c5664660ec48aaaf5826d79f280f
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
n_claims: 14
---

# overlapping-peak-deconvolution

## Summary

Fit series of 1D or pseudo-2D NMR spectra with MultiNMRFit multiplet signal models to deconvolve overlapping peaks and extract intensity, area, chemical shift, linewidth, and coupling constant(s) with per-parameter standard deviations.

## When to use

Use when 1D or pseudo-2D NMR spectra contain overlapping multiplet signals (e.g. a triplet and quadruplet, or overlapping ADP/ATP peaks) and accurate peak parameters must be recovered by model-based fitting rather than integration.

## When NOT to use

- The spectra contain only well-resolved, non-overlapping peaks where simple integration suffices and multiplet modeling adds no accuracy
- The data are not NMR spectra (e.g. LC-MS peaks or chromatograms), since MultiNMRFit's multiplet signal models are designed for 1D NMR fitting
- Higher-order (non first-order) multiplet patterns require user-defined models not yet available in the software

## Inputs

- Series of 1D NMR spectra (acquired individually or as a pseudo-2D experiment)
- Tabulated text files of synthetic or experimental spectra (e.g. triplet + quadruplet signal mixtures at varying overlap and noise)
- User-defined or built-in multiplet signal models per nucleus (1H, 13C, 15N, 31P, etc.)

## Outputs

- Estimated signal parameters (intensity, area, chemical shift, linewidth, coupling constant(s)) per signal per spectrum
- Uncertainty (standard deviation) for each fitted parameter
- Fitted curves overlaid on experimental spectra for visual inspection

## How to apply

Install MultiNMRFit with `pip install multinmrfit` (Python 3.8+) and load the spectra (individual 1D spectra, a pseudo-2D spectrum, or tabulated text files). Use the semi-automated peak picking and multiplicity definition to assign the appropriate multiplet signal model (e.g. triplet, quadruplet) to each signal of interest, and explicitly mark peaks that overlap with neighboring signals. Apply zero-order baseline correction, then fit the whole spectral series to estimate intensity, area, chemical shift, linewidth, and coupling constant(s) per signal. Judge success by visual inspection of fitted curves against experimental spectra — overlapping multiplets such as ADP/ATP must be captured — and by the reported standard deviations on each estimated parameter.

## Related tools

- **MultiNMRFit** (Open-source Python software used to pick peaks semi-automatically, define multiplicities, account for peak overlaps and zero-order baseline, and fit 1D/pseudo-2D NMR spectra with multiplet signal models) — https://github.com/NMRTeamTBI/MultiNMRFit/
- **pip** (Package installer used to install multinmrfit from GitHub/PyPI)

## Examples

```
pip install multinmrfit && nmrfit
```

## Evaluation signals

- Visual inspection shows fitted curves match experimental spectra and overlapping multiplets (e.g. ADP/ATP) are correctly captured
- Every fitted parameter is reported with a standard deviation, and uncertainties are reasonable across the series
- Fitted chemical shifts and coupling constants fall near expected multiplet positions for the assigned multiplicity (triplet/quadruplet)
- Fitting an entire series (e.g. 1024 pseudo-2D spectra) completes in roughly 3 hours, consistent with the reported throughput
- Concentration time courses derived from fitted areas behave physically (e.g. monotonic glycolysis metabolite consumption/production)

## Limitations

- Overlapping peaks must be explicitly flagged during setup; unmodeled overlaps degrade parameter accuracy
- Baseline is corrected to zero order only, so larger baselines drifts are not handled
- Software is validated for 1D and pseudo-2D NMR; performance on other modalities is out of scope
- Standard usage is via a graphical interface, which may limit full automation unless used as a Python library

## Evidence

- [abstract] abstract: "MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models"
- [readme] readme-overlap: "account for **overlaps** between peaks and **zero-order baseline correction**"
- [readme] readme-peakpicking: "**semi-automated analysis** for **peak picking** and **definition of multiplicity** for each signal"
- [methods] methods-install: "MultiNMRFit requires Python 3.8 or higher and run on all platforms (Windows, MacOS and Unix)"
