---
name: nmr-spectral-phase-correction
description: Use when processing a series of 1D-NMR spectra (e.g. metabolomics profiles)
  whose baselines show phasing/roll, and a manually phase-corrected reference spectrum
  is available to seed automated ph0/ph1 optimisation.
when_to_use_negative:
- The input is already a pre-processed feature table ready for multivariate analysis
  — phase correction operates on raw/reconstructed 1D spectra
- The spectra have a sweep width that does not cover the > 10.0 ppm and < -0.5 ppm
  baseline regions, so the objective function cannot be defined
- The data are 2D-NMR spectra, for which this baseline-edge ph0/ph1 optimisation scheme
  is not described
edam_topics:
- http://edamontology.org/topic_0523
tools:
- name: MetaboLabPy 0.9.66
  role: Provides the nmrData object and the automated phase correction algorithm for
    1D-NMR spectra series
  repo: https://github.com/ludwigc/MetaboLabPy
- name: scipy
  role: Powell's conjugate direction method (scipy.optimize) minimises the baseline-difference
    objective over ph0/ph1
- name: numpy
  role: Holds the spectral data arrays used in the phase optimisation objective
- name: pandas
  role: Collects the optimised ph0/ph1 values per spectrum and writes phase3_phases.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:b370196e83c64a528c5ce6632c6b6076b7ea06b9587348c3a9993bc8f34ab3dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MetaboLabPy 0.9.66
  - Protomix
  - Python
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# nmr-spectral-phase-correction

## Summary

Automatic phase correction of a series of 1D-NMR spectra in MetaboLabPy, where zero- and first-order phases (ph0/ph1) are initialised from a phase-corrected reference spectrum and then optimised per spectrum. It removes baseline roll across a series so spectra are ready for multivariate statistical analysis.

## When to use

Use when processing a series of 1D-NMR spectra (e.g. metabolomics profiles) whose baselines show phasing/roll, and a manually phase-corrected reference spectrum is available to seed automated ph0/ph1 optimisation.

## When NOT to use

- The input is already a pre-processed feature table ready for multivariate analysis — phase correction operates on raw/reconstructed 1D spectra
- The spectra have a sweep width that does not cover the > 10.0 ppm and < -0.5 ppm baseline regions, so the objective function cannot be defined
- The data are 2D-NMR spectra, for which this baseline-edge ph0/ph1 optimisation scheme is not described

## Inputs

- Series of 1D-NMR spectra loaded into metabolabpy's nmrData object
- A manually phase-corrected reference spectrum providing initial ph0/ph1 values
- numpy arrays holding the spectral data

## Outputs

- Phase-corrected series of 1D-NMR spectra with flat baselines
- Optimised ph0/ph1 values per spectrum, written to phase3_phases.csv via pandas

## How to apply

Load the series of 1D-NMR spectra into MetaboLabPy's nmrData object (metabolabpy 0.9.66). Initialise the zero-order (ph0) and first-order (ph1) phase values from the user's phase-corrected reference spectrum. Define the objective function as the absolute difference between the baseline signal in the left region (chemical shifts > 10.0 ppm) and the right region (< -0.5 ppm). Minimise this objective across the series using Powell's conjugate direction method from scipy.optimize, with numpy arrays holding the spectral data. Collect the optimised ph0/ph1 values per spectrum into a pandas table and write phase3_phases.csv.

## Related tools

- **MetaboLabPy 0.9.66** (Provides the nmrData object and the automated phase correction algorithm for 1D-NMR spectra series) — https://github.com/ludwigc/MetaboLabPy
- **scipy** (Powell's conjugate direction method (scipy.optimize) minimises the baseline-difference objective over ph0/ph1)
- **numpy** (Holds the spectral data arrays used in the phase optimisation objective)
- **pandas** (Collects the optimised ph0/ph1 values per spectrum and writes phase3_phases.csv)

## Examples

```
from scipy.optimize import minimize; res = minimize(baseline_diff_cost, x0=[ph0_ref, ph1_ref], args=(spec, ppm), method='Powell')
```

## Evaluation signals

- After optimisation, the baseline in both the > 10.0 ppm and < -0.5 ppm regions is flat and mutually consistent (minimal absolute difference)
- Each spectrum in the series has a recorded optimised ph0/ph1 pair in phase3_phases.csv (no missing rows)
- The absolute difference between left- and right-edge baseline regions has decreased relative to the pre-correction state
- Reference-spectrum ph0/ph1 initial values are propagated sensibly (optimised values near the reference phases for similar spectra)

## Limitations

- Requires a user-supplied phase-corrected reference spectrum to initialise ph0/ph1; quality depends on that reference
- Objective depends on baseline-only edge regions (> 10.0 ppm, < -0.5 ppm), so spectra without usable edge noise regions cannot be phased this way
- No changelog found for the package (per the article's index), so behaviour may vary across MetaboLabPy versions

## Evidence

- [abstract] This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms"
- [abstract] Complete workflow for pre-processing: "The software implements a complete workflow for NMR data pre-processing to prepare a series of 1D-NMR spectra for multi-variate statistical data analysis"
- [abstract] Python toolboxes: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc."
- [readme] Package purpose: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
- [abstract] Version and scope: "MetaboLabPy (version 0.9.66) is a free and open-source software package used to process 1D- and 2D-NMR spectra"
