---
name: synthetic-spectral-parameter-recovery
description: Use when you need to validate a spectral fitting workflow on data with
  known ground truth — e.g. synthetic 1D 1H NMR spectra with overlapping multiplets
  at low SNR — to quantify accuracy and precision of recovered parameters.
when_to_use_negative:
- The data are real experimental spectra with no ground truth — parameter recovery
  cannot be scored without known true values
- The spectra are 2D experiments that are not pseudo-2D series of 1D spectra, or the
  signals require signal models not built into MultiNMRFit and not user-definable
- The goal is peak detection or spectral preprocessing rather than quantitative multiplet
  parameter fitting
edam_operation: http://edamontology.org/operation_3860
edam_topics:
- http://edamontology.org/topic_0593
- http://edamontology.org/topic_3520
tools:
- name: MultiNMRFit
  role: 'Fitting engine: fits series of 1D spectra with built-in multiplet signal
    models, performs semi-automated peak picking, handles peak overlap and zero-order
    baseline correction, and reports parameter uncertainties'
  repo: https://github.com/NMRTeamTBI/MultiNMRFit/
- name: pip
  role: Package installer used to install multinmrfit from GitHub
- name: Python
  role: Runtime environment (Python 3.8+) in which MultiNMRFit is installed and run
    as a library or GUI
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf463
    title: 'MultiNMRFit: a software to fit 1D and pseudo-2D NMR spectra'
  source_content_hash: sha256:5fcce236fee7184c739ee2f43819c7269a82c0c63af838873b43649c88f4c085
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMRTeamTBI/MultiNMRFit
  tools:
  - MultiNMRFit
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_3860
  edam_topics:
  - http://edamontology.org/topic_0593
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# synthetic-spectral-parameter-recovery

## Summary

Benchmarking procedure for validating NMR fitting software by fitting synthetic 1D spectra containing overlapping multiplet signals (e.g. a triplet plus a quadruplet) across graded overlap and noise levels, using MultiNMRFit to recover known ground-truth parameters with uncertainty estimates.

## When to use

Use when you need to validate a spectral fitting workflow on data with known ground truth — e.g. synthetic 1D 1H NMR spectra with overlapping multiplets at low SNR — to quantify accuracy and precision of recovered parameters.

## When NOT to use

- The data are real experimental spectra with no ground truth — parameter recovery cannot be scored without known true values
- The spectra are 2D experiments that are not pseudo-2D series of 1D spectra, or the signals require signal models not built into MultiNMRFit and not user-definable
- The goal is peak detection or spectral preprocessing rather than quantitative multiplet parameter fitting

## Inputs

- Synthetic tabulated 1D NMR spectra (text files) containing overlapping triplet and quadruplet multiplet signals at defined overlap and signal-to-noise levels
- Ground-truth signal parameter values (intensity, area, chemical shift, linewidth, coupling constants) used to generate the synthetic spectra
- Peak positions and multiplicity definitions for each signal of interest

## Outputs

- Fitted parameter values (intensity, area, chemical shift, linewidth, coupling constant(s)) per signal per condition
- Standard deviation (uncertainty) on each fitted parameter
- fitted curve plots for visual inspection
- fitting_results.csv containing all fitted parameter values with their standard deviations

## How to apply

Install MultiNMRFit from GitHub with `pip install multinmrfit` under Python 3.8+, then load the synthetic tabulated spectra series (individual 1D or pseudo-2D text files) into the tool. Use its semi-automated peak picking and multiplicity definition to assign built-in multiplet signal models (e.g. triplet and quadruplet) to each signal of interest, letting the tool account for peak overlaps and zero-order baseline correction. Run the fit to estimate intensity, area, chemical shift, linewidth, and coupling constant(s) for each signal. Visually inspect the fitted curves and record the estimated uncertainty (standard deviation) on each parameter. Compare recovered values against the known synthetic ground truth across each overlap/noise condition; improved accuracy and precision from fitting whole multiplets (rather than isolated peaks) is the key rationale.

## Related tools

- **MultiNMRFit** (Fitting engine: fits series of 1D spectra with built-in multiplet signal models, performs semi-automated peak picking, handles peak overlap and zero-order baseline correction, and reports parameter uncertainties) — https://github.com/NMRTeamTBI/MultiNMRFit/
- **pip** (Package installer used to install multinmrfit from GitHub)
- **Python** (Runtime environment (Python 3.8+) in which MultiNMRFit is installed and run as a library or GUI)

## Examples

```
pip install multinmrfit; nmrfit
```

## Evaluation signals

- Fitted chemical shifts, intensities, linewidths and coupling constants of the triplet and quadruplet match the known synthetic ground truth within the reported standard deviations
- Recovery remains robust at low signal-to-noise ratio and significant peak-overlap conditions across the tested condition grid
- Each fitted parameter carries a reported standard deviation (uncertainty estimate), and fitting whole multiplets yields better accuracy/precision than baseline approaches
- Visual inspection of fitted curves shows the fitted multiplet model reproducing both overlapped peaks and the zero-order-corrected baseline
- All fitted values with standard deviations are written to fitting_results.csv without missing entries

## Limitations

- Validation relies on synthetic data; robustness on experimental spectra under identical overlap/noise conditions is a separate question
- No discussion/limitations section text was available in the extracted article, so published caveats could not be traced
- Requires Python 3.8 or higher; the pip install command in the README text references installing 'PhysioFit', indicating documentation copy-paste artifacts

## Evidence

- [abstract] Robust parameter estimation under low SNR and overlap: "MultiNMRFit extracts signal parameters (e.g. intensity, area, chemical shift, and coupling constants) by fitting the experimental spectra using built-in or user-defined signal models"
- [readme] Semi-automated setup, overlap handling, and uncertainty estimation: "semi-automated analysis for **peak picking** and **definition of multiplicity** for each signal, account for **overlaps** between peaks and **zero-order baseline correction**"
- [readme] Parameter set and uncertainty output: "estimation of several parameters for each signal of interest (**intensity**, **area**, **chemical shift**, **linewidth**, **coupling constant(s)**, etc)"
- [methods] Installation command: "Use `pip` to **install PhysioFit from GitHub**:

bash
$ pip install multinmrfit"
- [methods] Designed for series of 1D spectra: "MultiNMRFit is a scientific tool designed to extract quantitative information (chemical shifts, signal intensity, coupling constants, etc) by fitting a serie of 1D spectra"
