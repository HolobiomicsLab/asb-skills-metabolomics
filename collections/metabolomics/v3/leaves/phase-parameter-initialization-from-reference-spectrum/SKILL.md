---
name: phase-parameter-initialization-from-reference-spectrum
description: Use when a series of 1D-NMR spectra requires automated zero- and first-order
  phase correction and a phase-corrected reference spectrum is available to initialise
  ph0/ph1.
when_to_use_negative:
- The reference spectrum itself is not phase-corrected — seeded ph0/ph1 would start
  the Powell optimisation from a misleading point
- Spectra lack signal-free baseline regions beyond 10.0 ppm or below −0.5 ppm (e.g.
  resonances or truncation at the spectral edges), so the baseline-difference objective
  is meaningless
- Manual interactive phasing of a single 2D spectrum is needed rather than automated
  correction of a 1D series
tools:
- name: MetaboLabPy 0.9.66
  role: nmrData object holding the 1D-NMR spectral series and the reference-spectrum
    ph0/ph1 initialisation
  repo: https://github.com/ludwigc/MetaboLabPy
- name: scipy
  role: scipy.optimize provides Powell's conjugate direction method used to minimise
    the baseline-difference objective
- name: numpy
  role: numpy arrays hold the spectral data during objective evaluation
- name: pandas
  role: collects optimised ph0/ph1 values per spectrum into the phase3_phases.csv
    output table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.3390/metabo15010048
    title: MetaboLabPy—An Open-Source Software Package for Metabolomics NMR Data Processing
      and Metabolic Tracer Data Analysis
  source_content_hash: sha256:2a62dc89117d652a1dfda20139b087d600af9a41763d78c5faf733f81b7e9eca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ludwigc/MetaboLabPy
  tools:
  - MetaboLabPy 0.9.66
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# phase-parameter-initialization-from-reference-spectrum

## Summary

Seeds automated phase correction of a 1D-NMR spectral series with zero-order (ph0) and first-order (ph1) values taken from a phase-corrected reference spectrum, then optimises them per spectrum via MetaboLabPy's baseline-minimisation algorithm.

## When to use

Use when a series of 1D-NMR spectra requires automated zero- and first-order phase correction and a phase-corrected reference spectrum is available to initialise ph0/ph1.

## When NOT to use

- The reference spectrum itself is not phase-corrected — seeded ph0/ph1 would start the Powell optimisation from a misleading point
- Spectra lack signal-free baseline regions beyond 10.0 ppm or below −0.5 ppm (e.g. resonances or truncation at the spectral edges), so the baseline-difference objective is meaningless
- Manual interactive phasing of a single 2D spectrum is needed rather than automated correction of a 1D series

## Inputs

- Series of 1D-NMR spectra loaded into a metabolabpy nmrData object
- Phase-corrected reference spectrum supplying initial ph0/ph1 values

## Outputs

- Optimised ph0 and ph1 values per spectrum
- phase3_phases.csv table of per-spectrum phase parameters (pandas)
- Baseline-corrected spectral series with flat, mutually consistent edges

## How to apply

Load the series of 1D-NMR spectra into metabolabpy's nmrData object (MetaboLabPy 0.9.66). Initialise the zero-order (ph0) and first-order (ph1) phase values for each spectrum from the user-defined reference spectrum rather than from scratch, so the optimiser starts near a plausible solution. Define the objective as the absolute difference between the baseline signal in the left region (chemical shifts > 10.0 ppm) and the right region (chemical shifts < −0.5 ppm) of the spectra, held in numpy arrays. Minimise this objective across the series with Powell's conjugate direction method from scipy.optimize. Collect the optimised ph0/ph1 values per spectrum into a pandas table and write phase3_phases.csv.

## Related tools

- **MetaboLabPy 0.9.66** (nmrData object holding the 1D-NMR spectral series and the reference-spectrum ph0/ph1 initialisation) — https://github.com/ludwigc/MetaboLabPy
- **scipy** (scipy.optimize provides Powell's conjugate direction method used to minimise the baseline-difference objective)
- **numpy** (numpy arrays hold the spectral data during objective evaluation)
- **pandas** (collects optimised ph0/ph1 values per spectrum into the phase3_phases.csv output table)

## Examples

```
from scipy.optimize import minimize; res = minimize(lambda p: abs(baseline_left(p) - baseline_right(p)), x0=[ref_ph0, ref_ph1], method='Powell')
```

## Evaluation signals

- After optimisation, the baseline in the >10.0 ppm region is flat
- After optimisation, the baseline in the <−0.5 ppm region is flat
- Left- and right-edge baselines are mutually consistent (minimal absolute difference between the two regions)
- phase3_phases.csv contains one row per spectrum with optimised ph0 and ph1 values

## Limitations

- The method depends on signal-free edge regions (>10.0 ppm and <−0.5 ppm); peaks extending into these windows corrupt the objective
- Initialisation quality depends on the reference spectrum being correctly phase-corrected by the user
- No changelog found in the repository, so version-specific behaviour of the algorithm is not documented

## Evidence

- [abstract] This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms, and analysis of metabolic: "This includes a choice of algorithms for automated phase correction, segmental alignment, spectral scaling, variance stabilisation, export to various software platforms, and analysis of metabolic"
- [abstract] The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc: "The software is implemented in Python and uses numerous Python toolboxes, such as numpy, scipy, pandas, etc"
- [readme] Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis: "Python package to process 1D and 2D NMR spectroscopic data for metabolomics and tracer based metabolism analysis"
