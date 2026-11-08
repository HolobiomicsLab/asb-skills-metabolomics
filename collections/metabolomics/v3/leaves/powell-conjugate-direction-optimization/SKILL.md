---
name: powell-conjugate-direction-optimization
description: Use when a series of 1D-NMR spectra needs automated ph0/ph1 phase correction
  and a phase-corrected reference spectrum is available to seed the optimisation.
when_to_use_negative:
- Input is 2D-NMR or GC-MS / isotopomer data, which is handled by other MetaboLabPy/metabolabpytools
  routines (e.g. isotopomer analysis from HSQC multiplet and GC-MS data)
- A phase-corrected reference spectrum is not available, so initial ph0/ph1 values
  cannot be seeded from it
- Manual/interactive phase correction by visual inspection is required instead of
  automated correction
edam_operation: http://edamontology.org/operation_3257
edam_topics:
- http://edamontology.org/topic_0521
- http://edamontology.org/topic_3272
tools:
- name: MetaboLabPy 0.9.66
  role: Provides the nmrData object and the reference-spectrum baseline-based automatic
    phase correction algorithm
  repo: https://github.com/ludwigc/MetaboLabPy
- name: scipy
  role: Supplies Powell's conjugate direction method used to minimise the baseline-edge
    objective
  repo: https://github.com/ludwigc/metabolabpy
- name: numpy
  role: Holds the spectral data arrays used by the optimiser
  repo: https://github.com/ludwigc/metabolabpy
- name: pandas
  role: Collects optimised ph0/ph1 values into a table and writes phase3_phases.csv
  repo: https://github.com/ludwigc/metabolabpy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:a792f5d480b0bbc01420d09a6801c38c1df9f7520f5417e24a93f9818df419d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MetaboLabPy 0.9.66
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3257
  edam_topics:
  - http://edamontology.org/topic_0521
  - http://edamontology.org/topic_3272
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# powell-conjugate-direction-optimization

## Summary

Automatic phase correction of 1D-NMR spectra series in MetaboLabPy: zero- and first-order phases are optimised per spectrum by minimising the absolute difference between baseline edge regions (>10.0 ppm and <−0.5 ppm) using Powell's conjugate direction method from scipy.optimize.

## When to use

Use when a series of 1D-NMR spectra needs automated ph0/ph1 phase correction and a phase-corrected reference spectrum is available to seed the optimisation.

## When NOT to use

- Input is 2D-NMR or GC-MS / isotopomer data, which is handled by other MetaboLabPy/metabolabpytools routines (e.g. isotopomer analysis from HSQC multiplet and GC-MS data)
- A phase-corrected reference spectrum is not available, so initial ph0/ph1 values cannot be seeded from it
- Manual/interactive phase correction by visual inspection is required instead of automated correction

## Inputs

- Series of 1D-NMR spectra loaded into a metabolabpy nmrData object
- Phase-corrected reference spectrum (user-defined, providing initial ph0/ph1 values)

## Outputs

- Optimised ph0/ph1 phase values per spectrum
- phase3_phases.csv (pandas table of per-spectrum phases)
- Phase-corrected spectra series

## How to apply

Load the 1D-NMR spectra series into a metabolabpy nmrData object and initialise the zero-order (ph0) and first-order (ph1) phase values from the user-selected phase-corrected reference spectrum. Define the objective function as the absolute difference between the baseline signal in the left edge region (chemical shifts > 10.0 ppm) and the right edge region (chemical shifts < −0.5 ppm) of each spectrum. Minimise this objective across the series using Powell's conjugate direction method via scipy.optimize, with spectral data held in numpy arrays. Collect the optimised ph0/ph1 values per spectrum into a pandas table and write phase3_phases.csv.

## Related tools

- **MetaboLabPy 0.9.66** (Provides the nmrData object and the reference-spectrum baseline-based automatic phase correction algorithm) — https://github.com/ludwigc/MetaboLabPy
- **scipy** (Supplies Powell's conjugate direction method used to minimise the baseline-edge objective) — https://github.com/ludwigc/metabolabpy
- **numpy** (Holds the spectral data arrays used by the optimiser) — https://github.com/ludwigc/metabolabpy
- **pandas** (Collects optimised ph0/ph1 values into a table and writes phase3_phases.csv) — https://github.com/ludwigc/metabolabpy

## Examples

```
from scipy.optimize import minimize; res = minimize(baseline_edge_cost, x0=[ph0_ref, ph1_ref], args=(spec,), method='Powell')
```

## Evaluation signals

- After optimisation, the baseline in both the >10.0 ppm and <−0.5 ppm regions must be flat
- The absolute difference between the left (>10.0 ppm) and right (<−0.5 ppm) baseline regions is minimal after convergence
- A phase3_phases.csv table exists with one optimised ph0/ph1 pair per spectrum in the series
- Optimised spectra show consistent baseline offsets across the series rather than per-spectrum drift

## Limitations

- Requires a phase-corrected reference spectrum: ph0/ph1 values are initialised from the user-defined reference, so a poor reference biases the optimisation
- Objective relies on the spectral edges being baseline-only regions (>10.0 ppm and <−0.5 ppm); peaks in these regions would corrupt the cost function
- No changelog found for the software, so version-specific behaviour of the algorithm may be undocumented

## Evidence

- [abstract] Reference-spectrum baseline-based phase correction: "MetaboLabPy (version 0.9.66) is a free and open-source software package used to process 1D- and 2D-NMR spectra"
- [abstract] Toolbox basis: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc."
- [methods] Baseline-region objective and optimiser: [evidence span withheld — anchor status: not_found]
- [readme] Package availability: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
